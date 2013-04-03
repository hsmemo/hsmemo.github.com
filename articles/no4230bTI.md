---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp

### 名前(function name)
```
void ATTR ObjectMonitor::enter(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // The following code is ordered to check the most common cases first
	  // and to reduce RTS->RTO cache line upgrades on SPARC and IA32 processors.
	  Thread * const Self = THREAD ;
	  void * cur ;
	
  {- -------------------------------------------
  (1) ロックがアンロック状態かもしれないので, とりあえず CAS でロックを取れるかどうか試してみる.
      (具体的には, _owner を NULL からカレントスレッドに書き換えられるか試してみる)
      CAS が成功したら (= 本当にアンロック状態だったら), ここでリターン.
      ---------------------------------------- -}

	  cur = Atomic::cmpxchg_ptr (Self, &_owner, NULL) ;
	  if (cur == NULL) {
	     // Either ASSERT _recursions == 0 or explicitly set _recursions = 0.
	     assert (_recursions == 0   , "invariant") ;
	     assert (_owner      == Self, "invariant") ;
	     // CONSIDER: set or assert OwnerIsThread == 1
	     return ;
	  }
	
  {- -------------------------------------------
  (1) もしロックの所有者が自分であれば (= _owner がカレントスレッドであれば), 
      _recursions をインクリメントするだけでいい.
      この場合, ここでリターン.
      ---------------------------------------- -}

	  if (cur == Self) {
	     // TODO-FIXME: check for integer overflow!  BUGID 6557169.
	     _recursions ++ ;
	     return ;
	  }
	
  {- -------------------------------------------
  (1) 自分がロックを取得して stack-locked 状態にしたオブジェクトかどうかを調べる.
        (注: stack-locked 状態のロックが inflate された直後は, _owner に正しい値が入っていない.
         この場合には, スレッドではなく, 対応する BasicObjectLock が入っている.
         See: ObjectSynchronizer::inflate())
      もしそうであれば, (単に ObjectMonitor オブジェクトのフィールドに正しい値が入っていないだけなので)
      ObjectMonitor の _owner フィールド等の設定を行い, ここでリターン.
    
      (なおこの場合は, 意味的には自分が2回目のロックを取ったことになるので, 
       _recursions は 1 とする.)
      ---------------------------------------- -}

	  if (Self->is_lock_owned ((address)cur)) {
	    assert (_recursions == 0, "internal state error");
	    _recursions = 1 ;
	    // Commute owner from a thread-specific on-stack BasicLockObject address to
	    // a full-fledged "Thread *".
	    _owner = Self ;
	    OwnerIsThread = 1 ;
	    return ;
	  }
	
  {- -------------------------------------------
  (1) (これ以降は, 本当に contention しているケース)
      ---------------------------------------- -}

	  // We've encountered genuine contention.

  {- -------------------------------------------
  (1) カレントスレッドの _Stalled フィールドに値をセットしておく.
      (これは, Adaptive Spinnning 中に使用される模様.
       See: ObjectMonitor::NotRunnable())
      ---------------------------------------- -}

	  assert (Self->_Stalled == 0, "invariant") ;
	  Self->_Stalled = intptr_t(this) ;
	
  {- -------------------------------------------
  (1) とりあえず TrySpin() でスピンロックしてみる. スピンロックが成功したらここでリターン.
      (ただし, この最適化は Knob_SpinEarly がセットされている場合にのみ行われる)
  
      (なお, TrySpin とは ObjectMonitor::TrySpin_VaryDuration() のこと.
       See: Adaptive Spinning)
      ---------------------------------------- -}

	  // Try one round of spinning *before* enqueueing Self
	  // and before going through the awkward and expensive state
	  // transitions.  The following spin is strictly optional ...
	  // Note that if we acquire the monitor from an initial spin
	  // we forgo posting JVMTI events and firing DTRACE probes.
	  if (Knob_SpinEarly && TrySpin (Self) > 0) {
	     assert (_owner == Self      , "invariant") ;
	     assert (_recursions == 0    , "invariant") ;
	     assert (((oop)(object()))->mark() == markOopDesc::encode(this), "invariant") ;
	     Self->_Stalled = 0 ;
	     return ;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (_owner != Self          , "invariant") ;
	  assert (_succ  != Self          , "invariant") ;
	  assert (Self->is_Java_thread()  , "invariant") ;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread * jt = (JavaThread *) Self ;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (!SafepointSynchronize::is_at_safepoint(), "invariant") ;
	  assert (jt->thread_state() != _thread_blocked   , "invariant") ;
	  assert (this->object() != NULL  , "invariant") ;
	  assert (_count >= 0, "invariant") ;
	
  {- -------------------------------------------
  (1) これから待機状態に入るので, 待機対象の ObjectMonitor の 'reference count' を増加させておく.
      (これは, Safepoint 時に deflate されるのを防ぐ効果がある.
       See: ObjectSynchronizer::deflate_idle_monitors())
      ---------------------------------------- -}

	  // Prevent deflation at STW-time.  See deflate_idle_monitors() and is_busy().
	  // Ensure the object-monitor relationship remains stable while there's contention.
	  Atomic::inc_ptr(&_count);
	
  {- -------------------------------------------
  (1) JavaThreadBlockedOnMonitorEnterState で, JavaThread の状態を変更しておく.
  
      なお, JavaThreadBlockedOnMonitorEnterState には(プロファイル情報の記録)という役割もある.
      (See: [here](no21146np.html) for details)
      ---------------------------------------- -}

	  { // Change java thread status to indicate blocked on monitor enter.
	    JavaThreadBlockedOnMonitorEnterState jtbmes(jt, this);
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	    DTRACE_MONITOR_PROBE(contended__enter, this, object(), jt);

  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	    if (JvmtiExport::should_post_monitor_contended_enter()) {
	      JvmtiExport::post_monitor_contended_enter(jt, this);
	    }
	
  {- -------------------------------------------
  (1) ThreadBlockInVM 及び OSThreadContendState で, JavaThread/OSThread の状態を変更しておく.
      ---------------------------------------- -}

	    OSThreadContendState osts(Self->osthread());
	    ThreadBlockInVM tbivm(jt);
	
  {- -------------------------------------------
  (1) これから待機状態に入るので, 待機対象の ObjectMonitor をカレントスレッド内に記録しておく.
      (この情報は JVMTI や JMM の処理で利用されている模様. #TODO)
      ---------------------------------------- -}

	    Self->set_current_pending_monitor(this);
	
  {- -------------------------------------------
  (1) ObjectMonitor::EnterI() を呼び出して, ロックを確保する 
      (というか, ロックが確保できるまで眠りにつく).
      
      なお, 眠っている間に suspend された場合のために, 
      ObjectMonitor::EnterI() から返ってきた後で 
      ObjectMonitor::ExitSuspendEquivalent() による確認を行っている.
    
      suspend されていた場合は, (せっかくロックは取れたがこのまま実行を続けるとおかしなことになるので)
      ObjectMonitor::exit() でロックを解放し, 
      さらに JavaThread::java_suspend_self() でサスペンドが解除されるまで眠りにつく.
      サスペンドが解除された後は, 再度 ObjectMonitor::EnterI() を呼び出して, ロック確保を行う.
      (for ループになっているのはこのため. 
       suspend されなかった場合は, この for ループは1回しか実行されない.
       なお, コメントによるとループじゃない形に書き直したい模様.)
  
  
      #TODO  JavaThread::set_suspend_equivalent() はどういう意味がある??
      ---------------------------------------- -}

	    // TODO-FIXME: change the following for(;;) loop to straight-line code.
	    for (;;) {
	      jt->set_suspend_equivalent();
	      // cleared by handle_special_suspend_equivalent_condition()
	      // or java_suspend_self()
	
	      EnterI (THREAD) ;
	
	      if (!ExitSuspendEquivalent(jt)) break ;
	
	      //
	      // We have acquired the contended monitor, but while we were
	      // waiting another thread suspended us. We don't want to enter
	      // the monitor while suspended because that would surprise the
	      // thread that suspended us.
	      //
	          _recursions = 0 ;
	      _succ = NULL ;
	      exit (Self) ;
	
	      jt->java_suspend_self();
	    }

  {- -------------------------------------------
  (1) 待機状態は終わったので, 待機対象の ObjectMonitor の情報をクリアしておく.
      (この情報は JVMTI や JMM の処理で利用されている模様. #TODO)
      ---------------------------------------- -}

	    Self->set_current_pending_monitor(NULL);
	  }
	
  {- -------------------------------------------
  (1) 待機状態は終わったので, 待機対象の ObjectMonitor の 'reference count' を減少させておく.
      (これ以降は, Safepoint 時に deflate されても問題ない.
       See: ObjectSynchronizer::deflate_idle_monitors())
      ---------------------------------------- -}

	  Atomic::dec_ptr(&_count);
	  assert (_count >= 0, "invariant") ;

  {- -------------------------------------------
  (1) カレントスレッドの _Stalled フィールドの値もクリアしておく.
      (これは, Adaptive Spinnning 中に使用される模様.
       See: ObjectMonitor::NotRunnable())
      ---------------------------------------- -}

	  Self->_Stalled = 0 ;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Must either set _recursions = 0 or ASSERT _recursions == 0.
	  assert (_recursions == 0     , "invariant") ;
	  assert (_owner == Self       , "invariant") ;
	  assert (_succ  != Self       , "invariant") ;
	  assert (((oop)(object()))->mark() == markOopDesc::encode(this), "invariant") ;
	
  {- -------------------------------------------
  (1) (カレントスレッドは, 無事にロックを確保し VM mode に復帰してきたので, 
       以降で JVMTI や DTrace, jvmstat に対してその旨を報告する.
       ただし, この処理はロックを確保している間に行われるため, 
       critical section を増加させてしまうという影響がある.
       
       他のやり方としては, ここでは統計情報を集めるだけにしておき, 
       報告するのは後回しにする, という方法も考えられる.
       例えば, 次回ロック競合が起きた際に, ロックを確保する前に報告処理を行うとか.)
      ---------------------------------------- -}

	  // The thread -- now the owner -- is back in vm mode.
	  // Report the glorious news via TI,DTrace and jvmstat.
	  // The probe effect is non-trivial.  All the reportage occurs
	  // while we hold the monitor, increasing the length of the critical
	  // section.  Amdahl's parallel speedup law comes vividly into play.
	  //
	  // Another option might be to aggregate the events (thread local or
	  // per-monitor aggregation) and defer reporting until a more opportune
	  // time -- such as next time some thread encounters contention but has
	  // yet to acquire the lock.  While spinning that thread could
	  // spinning we could increment JVMStat counters, etc.
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_MONITOR_PROBE(contended__entered, this, object(), jt);

  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  if (JvmtiExport::should_post_monitor_contended_entered()) {
	    JvmtiExport::post_monitor_contended_entered(jt, this);
	  }

  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: UsePerfData) (See: sun.rt._sync_ContendedLockAttempts)
      ---------------------------------------- -}

	  if (ObjectMonitor::_sync_ContendedLockAttempts != NULL) {
	     ObjectMonitor::_sync_ContendedLockAttempts->inc() ;
	  }
	}
	
```


