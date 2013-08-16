---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
void Parker::park(bool isAbsolute, jlong time) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) もし _counter の値が 1 であれば (= 既に unpark() が呼ばれていれば), 
      単に _counter を 0 にするだけでいい.
      ここでリターン.
  
      (なお, ここから先が Parker で保護された critical section ということになるので, 
       OrderAccess::fence() も張っておく)
      ---------------------------------------- -}

	  // Optional fast-path check:
	  // Return immediately if a permit is available.
	  if (_counter > 0) {
	      _counter = 0 ;
	      OrderAccess::fence();
	      return ;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Optional fast-exit: Check interrupt before trying to wait
	  Thread* thread = Thread::current();
	  assert(thread->is_Java_thread(), "Must be JavaThread");
	  JavaThread *jt = (JavaThread *)thread;

  {- -------------------------------------------
  (1) 処理対象のスレッドに対して java.lang.Thread.interrupt() が呼ばれていた場合は, 
      待機する必要は無いので, ここでリターン.
      (interrupt されたかどうかは後でもチェックしているのでこの処理はなくてもいいが, 
       ここでチェックに引っかかれば ThreadBlockInVM による状態遷移が必要なくなる, という最適化)
      ---------------------------------------- -}

	  if (Thread::is_interrupted(thread, false)) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) 引数で指定された待ち時間(time)をチェックする. 以下の場合はここでリターン.
      * 指定された待ち時間が負値の場合
      * 指定された待ち時間が絶対時間であり, その時間が現在時刻よりも古い場合
      ---------------------------------------- -}

	  // First, demultiplex/decode time arguments
	  timespec absTime;
	  if (time < 0 || (isAbsolute && time == 0) ) { // don't wait at all
	    return;
	  }

  {- -------------------------------------------
  (1) unpackTime() を呼んで, 指定された待ち時間の値(time)を絶対時刻に調整しておく.
      ---------------------------------------- -}

	  if (time > 0) {
	    // Warning: this code might be exposed to the old Solaris time
	    // round-down bugs.  Grep "roundingFix" for details.
	    unpackTime(&absTime, isAbsolute, time);
	  }
	
  {- -------------------------------------------
  (1) (コメントによると, デッドロックに注意とのこと.
       Parker の _mutex を握った状態で Threads_lock でブロックしてはいけない.
       Safepoint が開始されている場合には, ThreadBlockInVM のコンストラクタやデストラクタでは 
       Threads_lock を確保するかもしれない.)
      ---------------------------------------- -}

	  // Enter safepoint region
	  // Beware of deadlocks such as 6317397.
	  // The per-thread Parker:: _mutex is a classic leaf-lock.
	  // In particular a thread must never block on the Threads_lock while
	  // holding the Parker:: mutex.  If safepoints are pending both the
	  // the ThreadBlockInVM() CTOR and DTOR may grab Threads_lock.

  {- -------------------------------------------
  (1) ThreadBlockInVM で JavaThread の状態を変更しておく.
      ---------------------------------------- -}

	  ThreadBlockInVM tbivm(jt);
	
  {- -------------------------------------------
  (1) 処理対象のスレッドに対して java.lang.Thread.interrupt() が呼ばれていた場合は, ここでリターンする.
      そうでなければ, os::Solaris::mutex_trylock() で _mutex を取得する.
  
      (なお, _mutex のロック処理が unpark() 処理と競合した場合 (= os::Solaris::mutex_trylock() が非ゼロを返した場合) は, 
       ここでリターンする.)
      ---------------------------------------- -}

	  // Don't wait if cannot get lock since interference arises from
	  // unblocking.  Also. check interrupt before trying wait
	  if (Thread::is_interrupted(thread, false) ||
	      os::Solaris::mutex_trylock(_mutex) != 0) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int status ;
	
  {- -------------------------------------------
  (1) もし _counter の値が 1 であれば (= 既に unpark() が呼ばれていれば), 
      単に _counter を 0 にするだけでいい.
      os::Solaris::mutex_unlock() で _mutex を解放した後, ここでリターン.
  
      (なお, ここから先が Parker で保護された critical section ということになるので, 
       OrderAccess::fence() も張っておく)
      ---------------------------------------- -}

	  if (_counter > 0)  { // no wait needed
	    _counter = 0;
	    status = os::Solaris::mutex_unlock(_mutex);
	    assert (status == 0, "invariant") ;
	    OrderAccess::fence();
	    return;
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // Don't catch signals while blocked; let the running threads have the signals.
	  // (This allows a debugger to break into the running thread.)
	  sigset_t oldsigs;
	  sigset_t* allowdebug_blocked = os::Solaris::allowdebug_blocked_signals();
	  thr_sigsetmask(SIG_BLOCK, allowdebug_blocked, &oldsigs);
	#endif
	
  {- -------------------------------------------
  (1) OSThreadWaitState で OSThread の状態を変更した後, 
      os::Solaris::cond_wait() または os::Solaris::cond_timedwait() を呼んで 
      誰かが unpark() してくれるまで眠りにつく.
  
      (指定された待ち時間が 0 (= 無制限) であれば os::Solaris::cond_wait(), 
       そうでなければ os::Solaris::cond_timedwait() を呼び出す.)
    
      (なお, ClearFPUAtPark オプションが指定されている場合は... #TODO)
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	  OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
	  jt->set_suspend_equivalent();
	  // cleared by handle_special_suspend_equivalent_condition() or java_suspend_self()
	
	  // Do this the hard way by blocking ...
	  // See http://monaco.sfbay/detail.jsf?cr=5094058.
	  // TODO-FIXME: for Solaris SPARC set fprs.FEF=0 prior to parking.
	  // Only for SPARC >= V8PlusA
	#if defined(__sparc) && defined(COMPILER2)
	  if (ClearFPUAtPark) { _mark_fpu_nosave() ; }
	#endif
	
	  if (time == 0) {
	    status = os::Solaris::cond_wait (_cond, _mutex) ;
	  } else {
	    status = os::Solaris::cond_timedwait (_cond, _mutex, &absTime);
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Note that an untimed cond_wait() can sometimes return ETIME on older
	  // versions of the Solaris.
	  assert_status(status == 0 || status == EINTR ||
	                status == ETIME || status == ETIMEDOUT,
	                status, "cond_timedwait");
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  thr_sigsetmask(SIG_SETMASK, &oldsigs, NULL);
	#endif

  {- -------------------------------------------
  (1) _counter の値を 0 に戻し, _mutex を解放する.
      (ここまでが _mutex で排他された処理)
      ---------------------------------------- -}

	  _counter = 0 ;
	  status = os::Solaris::mutex_unlock(_mutex);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_status(status == 0, status, "mutex_unlock") ;
	
  {- -------------------------------------------
  (1) 寝ている間に java.lang.Thread.suspend() で suspend 状態にされたかもしれないので, 
      目が覚めた後に JavaThread::handle_special_suspend_equivalent_condition() でチェックしておく.
      もし suspend されていれば, JavaThread::java_suspend_self() でサスペンドが解除されるまで眠りにつく.
      ---------------------------------------- -}

	  // If externally suspended while waiting, re-suspend
	  if (jt->handle_special_suspend_equivalent_condition()) {
	    jt->java_suspend_self();
	  }

  {- -------------------------------------------
  (1) (なお, ここから先が Parker で保護された critical section ということになるので, 
       OrderAccess::fence() を張っておく)
      ---------------------------------------- -}

	  OrderAccess::fence();
	}
	
```


