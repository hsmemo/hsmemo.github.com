---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp
### 説明(description)

```
// Roll all threads forward to a safepoint and suspend them all
```

### 名前(function name)
```
void SafepointSynchronize::begin() {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Thread* myThread = Thread::current();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(myThread->is_VM_thread(), "Only VM thread may execute a safepoint");
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録) 
      (See: SafepointSynchronize::update_statistics_on_sync_end(), SafepointSynchronize::begin_statistics())
      ---------------------------------------- -}

	  if (PrintSafepointStatistics || PrintSafepointStatisticsTimeout > 0) {
	    _safepoint_begin_time = os::javaTimeNanos();
	    _ts_of_current_safepoint = tty->time_stamp().seconds();
	  }
	
  {- -------------------------------------------
  (1) #TODO
      (CMS や G1GC 向けの処理.
       concurrent thread を停止させる処理??)
      ---------------------------------------- -}

	#ifndef SERIALGC
	  if (UseConcMarkSweepGC) {
	    // In the future we should investigate whether CMS can use the
	    // more-general mechanism below.  DLD (01/05).
	    ConcurrentMarkSweepThread::synchronize(false);
	  } else if (UseG1GC) {
	    ConcurrentGCThread::safepoint_synchronize();
	  }
	#endif // SERIALGC
	
  {- -------------------------------------------
  (1) Safepoint 処理中にスレッドが新たに作られたり破棄されたりしないよう, ここで Threads_lock をロックしておく.
      (このロックは SafepointSynchronize::end() 内で解除される)
      ---------------------------------------- -}

	  // By getting the Threads_lock, we assure that no threads are about to start or
	  // exit. It is released again in SafepointSynchronize::end().
	  Threads_lock->lock();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert( _state == _not_synchronized, "trying to safepoint synchronize with wrong state");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int nof_threads = Threads::number_of_threads();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceSafepoint) {
	    tty->print_cr("Safepoint synchronization initiated. (%d)", nof_threads);
	  }
	
  {- -------------------------------------------
  (1) (DTrace のフック点) 及び 
      (プロファイル情報の記録)(JMM 用) ("sun.rt.safepoints", "sun.rt.applicationTime")
  
      (See: RuntimeService::record_safepoint_begin())
      ---------------------------------------- -}

	  RuntimeService::record_safepoint_begin();
	
  {- -------------------------------------------
  (1) (以下の処理は Safepoint_lock で排他した状態で行う)
      ---------------------------------------- -}

	  {
	  MutexLocker mu(Safepoint_lock);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Set number of threads to wait for, before we initiate the callbacks
	  _waiting_to_block = nof_threads;
	  TryingToBlock     = 0 ;
	  int still_running = nof_threads;
	
	  // Save the starting time, so that it can be compared to see if this has taken
	  // too long to complete.
	  jlong safepoint_limit_time;
	  timeout_error_printed = false;
	
  {- -------------------------------------------
  (1) (プロファイル情報取得用の初期化処理)
      SafepointSynchronize::deferred_initialize_stat() を呼んで, 
      (まだ初期化が終わっていなければ)プロファイル情報用の変数の初期化を行う.
      ---------------------------------------- -}

	  // PrintSafepointStatisticsTimeout can be specified separately. When
	  // specified, PrintSafepointStatistics will be set to true in
	  // deferred_initialize_stat method. The initialization has to be done
	  // early enough to avoid any races. See bug 6880029 for details.
	  if (PrintSafepointStatistics || PrintSafepointStatisticsTimeout > 0) {
	    deferred_initialize_stat();
	  }
	
  {- -------------------------------------------
  (1) (これ以降の処理で, HotSpot を Safepoint 状態に移行させる. 
       Java thread の実行状態に応じて, いくつかの止め方が存在する.
        
       1. インタープリタ上でコードを実行中のスレッドの場合:
          dispatchTable を変更することで safepoint チェック用のコードを実行させる
          (インタープリタが次のバイトコードへ遷移する際に停止)
       
          (なおこれは Template Interpreter の場合に必要となる処理. C++ Interpreter の場合は特に何もする必要は無い)
  
       2. ネイティブコードを実行中のスレッドの場合:
          ネイティブコードから HotSpot 内に戻ってきたときに SafepointSynchronize::_state をチェックしている.
          HotSpot 内に戻ってこなければ, チェックもないので停止されないが, 
          ネイティブコード実行中のスレッドに付いては VM Thread も停止するまで待たないことにしている.
     
          なお, SafepointSynchronize::_state の変更/確認と 
          各 JavaThread の state の変更/確認の順序は重要であり, 
          memory の consistency が保たれていないと破綻する.
     
          そのため, VMThread が SafepointSynchronize::_state を変更する際には
          (MP 環境なら) memory barrier を発行している.
          しかし, JavaThread 側については, オーバーヘッドが大きいので memory barrier を張っていない.
  
          代わりに, VM Thread が実行する SafepointSynchronize::begin() 内で 
          memory barrier を張った後に, 特定のメモリページの保護属性を変更するという処理を行っている.
          (See: os::serialize_thread_states()).
          各 Java Thread は, state 変更後に memory barrier の代わりに上記ページに write を行う.
  
       3. JIT 生成したコードを実行中のスレッドの場合:
          JIT 生成したコードでは, 定期的に Safepoint Polling page にメモリアクセスするようになっているので, 
          このページをアクセス禁止にして止める.
          (See: os::make_polling_page_unreadable())
  
       4. 現在ブロックされているスレッドの場合:
          Safepoint が終わるまでブロックされたままにする.
  
       5. VM 内のコードを実行中 (もしくは状態を遷移中) のスレッドの場合:
          新しい state に transition しようとした際に, 
          transition コード中のチェックコードがスレッドを止める.        )
      ---------------------------------------- -}

	  // Begin the process of bringing the system to a safepoint.
	  // Java threads can be in several different states and are
	  // stopped by different mechanisms:
	  //
	  //  1. Running interpreted
	  //     The interpeter dispatch table is changed to force it to
	  //     check for a safepoint condition between bytecodes.
	  //  2. Running in native code
	  //     When returning from the native code, a Java thread must check
	  //     the safepoint _state to see if we must block.  If the
	  //     VM thread sees a Java thread in native, it does
	  //     not wait for this thread to block.  The order of the memory
	  //     writes and reads of both the safepoint state and the Java
	  //     threads state is critical.  In order to guarantee that the
	  //     memory writes are serialized with respect to each other,
	  //     the VM thread issues a memory barrier instruction
	  //     (on MP systems).  In order to avoid the overhead of issuing
	  //     a memory barrier for each Java thread making native calls, each Java
	  //     thread performs a write to a single memory page after changing
	  //     the thread state.  The VM thread performs a sequence of
	  //     mprotect OS calls which forces all previous writes from all
	  //     Java threads to be serialized.  This is done in the
	  //     os::serialize_thread_states() call.  This has proven to be
	  //     much more efficient than executing a membar instruction
	  //     on every call to native code.
	  //  3. Running compiled Code
	  //     Compiled code reads a global (Safepoint Polling) page that
	  //     is set to fault if we are trying to get to a safepoint.
	  //  4. Blocked
	  //     A thread which is blocked will not be allowed to return from the
	  //     block condition until the safepoint operation is complete.
	  //  5. In VM or Transitioning between states
	  //     If a Java thread is currently running in the VM or transitioning
	  //     between states, the safepointing code will wait for the thread to
	  //     block itself when it attempts transitions to a new state.
	  //

  {- -------------------------------------------
  (1) SafepointSynchronize::_state を _synchronizing に変更する.
      上に書かれているように順序も大事なので, OrderAccess::fence() でメモリバリアも張っている.
  
      (SafepointSynchronize::_state を変更したことで, 
       SafepointSynchronize::is_synchronizing() が true を返すようになる.
       See: SafepointSynchronize::is_synchronizing())
      ---------------------------------------- -}

	  _state            = _synchronizing;
	  OrderAccess::fence();
	
  {- -------------------------------------------
  (1) os::serialize_thread_states() を呼んで, serialize page のメモリプロテクションを変化させる.
      (See: MacroAssembler::serialize_memory(), InterfaceSupport::serialize_memory())
  
      (これは上記の区分だと 2. に当たる処理)
  
      (とはいえ, JavaThread 側では, 特に native に限定せず serialize page を使っている模様.
       See: ThreadStateTransition::transition())
      ---------------------------------------- -}

	  // Flush all thread states to memory
	  if (!UseMembar) {
	    os::serialize_thread_states();
	  }
	
  {- -------------------------------------------
  (1) Interpreter::notice_safepoints() (またはそれをサブクラスがオーバーライドしたもの) を呼んで, 
      インタープリタの dispatch table を Safepoint 用のものに置き換える.
    
      (これは上記の区分だと 1. に当たる処理)
  
      (なお, 上述の通り. これは Template Interpreter の場合に必要となる処理.
      C++ Interpreter の場合は特に何もする必要は無い)
      ---------------------------------------- -}

	  // Make interpreter safepoint aware
	  Interpreter::notice_safepoints();
	
  {- -------------------------------------------
  (1) os::make_polling_page_unreadable() を呼んで, 
      Safepoint Polling page をアクセス不可にしておく.
      (ただし, UseCompilerSafepoints オプションが指定されていない場合は, この処理は行わない.
       また, DeferPollingPageLoopCount オプションが 0 以上の値の場合は, この処理は後で行う.)
  
      (これは上記の区分だと 3. に当たる処理)
      ---------------------------------------- -}

	  if (UseCompilerSafepoints && DeferPollingPageLoopCount < 0) {
	    // Make polling safepoint aware
	    guarantee (PageArmed == 0, "invariant") ;
	    PageArmed = 1 ;
	    os::make_polling_page_unreadable();
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Consider using active_processor_count() ... but that call is expensive.
	  int ncpus = os::processor_count() ;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  for (JavaThread *cur = Threads::first(); cur != NULL; cur = cur->next()) {
	    assert(cur->safepoint_state()->is_running(), "Illegal initial state");
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) (トレース出力用の処理)
      ---------------------------------------- -}

	  if (SafepointTimeout)
	    safepoint_limit_time = os::javaTimeNanos() + (jlong)SafepointTimeoutDelay * MICROUNITS;
	
  {- -------------------------------------------
  (1) (全ての JavaThread の ThreadSafepointState が running 以外の状態に変わるまで, 
       以下の while ループ内で待機する)
      ---------------------------------------- -}

	  // Iterate through all threads until it have been determined how to stop them all at a safepoint
	  unsigned int iterations = 0;
	  int steps = 0 ;
	  while(still_running > 0) {

    {- -------------------------------------------
  (1.1) ThreadSafepointState が running のままの JavaThread の数を数える (以下の still_running).
  
        (正確には, still_running は最初は「全JavaThread数」に設定されている (上の still_running の定義部参照).
         その後, running のままの JavaThread について 
         ThreadSafepointState::examine_state_of_thread() を呼んでみて
         running 以外に変わるかどうかを調べる.
         変わったら, その分だけ still_running を減少させる.)
    
        (なおコメントでは, 
         still_running を変更した際には steps も少し減らす調整をしたらどうか, 
         と提案されている)
  
  
        (ついでに, (トレース出力)も出している)
        ---------------------------------------- -}

	    for (JavaThread *cur = Threads::first(); cur != NULL; cur = cur->next()) {
	      assert(!cur->is_ConcurrentGC_thread(), "A concurrent GC thread is unexpectly being suspended");
	      ThreadSafepointState *cur_state = cur->safepoint_state();
	      if (cur_state->is_running()) {
	        cur_state->examine_state_of_thread();
	        if (!cur_state->is_running()) {
	           still_running--;
	           // consider adjusting steps downward:
	           //   steps = 0
	           //   steps -= NNN
	           //   steps >>= 1
	           //   steps = MIN(steps, 2000-100)
	           //   if (iterations != 0) steps -= NNN
	        }
	        if (TraceSafepoint && Verbose) cur_state->print();
	      }
	    }
	
    {- -------------------------------------------
  (1.1) (プロファイル情報の記録)
        (See: SafepointSynchronize::begin_statistics())
    
        (なお, この記録処理を実行するのは最初の1回だけ(= iterations が 0 の時だけ))
        ---------------------------------------- -}

	    if (PrintSafepointStatistics && iterations == 0) {
	      begin_statistics(nof_threads, still_running);
	    }
	
    {- -------------------------------------------
  (1.1) ThreadSafepointState が running のスレッドが残っていれば, 
        以下の if ブロック内で, 待機して少し時間を潰す.
        ---------------------------------------- -}

	    if (still_running > 0) {

      {- -------------------------------------------
  (1.1.1) (トレース出力)
    
          (なお, デバッグ用の DieOnSafepointTimeout オプションが指定されている場合は, 
           Safepoint 処理に時間が掛かりすぎている場合, ここで強制終了.
           See: SafepointSynchronize::print_safepoint_timeout())
          ---------------------------------------- -}

	      // Check for if it takes to long
	      if (SafepointTimeout && safepoint_limit_time < os::javaTimeNanos()) {
	        print_safepoint_timeout(_spinning_timeout);
	      }
	
      {- -------------------------------------------
  (1.1.1) (スピンして待つのがいいか CPU を手放した方がいいのかは一長一短.
  
           さらに面倒なことに, yield() は多くの環境では想定通りに動かない.
           そのため, 何度か失敗したら yield_all() にフォールバックさせている.
           yield_all() は, プラットフォームによっては短時間の sleep() として実装されている.
           典型的な OS では, sleep 時間を 10 msec 単位で切り上げて処理するので, 
           sleep すると safepoint 処理の時間は長くなりうる.
           (Bug 6415670 も参照).
           
           なお, スピンするかどうかを決める上では,
           デフォルトだと VMThread が mutator より高い優先度に設定されていることにも注意.
           
           また, Windows XP の SwitchThreadTo() は Sleep(0) とは異なる挙動になることにも注意.
           Sleep(0) は優先度が低いスレッドに yield しないが, SwitchThreadTo() はする.
           
           スピンについては, synchronizer.cpp 内のコメントも参照のこと.
    
  
           将来的には以下のようにしたい.
           1. safepoint 処理で, 無期限にスピンする可能性があるのは止めたい.
              これが難しいのは, (JNI call-out 等で) JVM の外に出て行くスレッドは state フィールドを変更するだけなので, 
              VMThread 側が polling(spin) して検出してやらないといけない点.
           2. スピン中に何か意味のあることを出来るようにしたい. 
              例えば GC 処理のための safepoint なら, この段階でスタック等を調査してしまうとか.
           3. Solaris なら, schedctl で still_running なスレッドの状態を確認する.
              全員が ONPROC なら sleep したり yield したりする必要は無い.
           4. YieldTo() するときは, still_running だけど OFFPROC なスレッドを狙いたい.
           5. CPU がサチっているかどうかを確認し, サチっていなければスピンする.
           6. still_running なスレッドが safepiont に到達したら, VMThread を起床させる.
              (ただし, VMThread 側では (JNI call-out 等をするスレッドのために) poll は必要)
           7. ループ回数ではなく, safepoint 処理開始時からの経過時間 (time-since-begin) に基づく処理にする.
           8. スピンする時間を CPU 数に応じて変えるようにしたい. 
                Spin = (((ncpus-1) * M) + K) + F(still_running)
              あるいは, ループ回数を数える代わりに, 上の still_running を計算するループで調べたスレッド数を数えるとか.
           9. Windows では, SwitchThreadTo() の返値を見て, 
              適切な挙動(スピン or SwitchThreadTo() or Sleep(N)) を選ぶようにしたい.
          ---------------------------------------- -}

	      // Spin to avoid context switching.
	      // There's a tension between allowing the mutators to run (and rendezvous)
	      // vs spinning.  As the VM thread spins, wasting cycles, it consumes CPU that
	      // a mutator might otherwise use profitably to reach a safepoint.  Excessive
	      // spinning by the VM thread on a saturated system can increase rendezvous latency.
	      // Blocking or yielding incur their own penalties in the form of context switching
	      // and the resultant loss of $ residency.
	      //
	      // Further complicating matters is that yield() does not work as naively expected
	      // on many platforms -- yield() does not guarantee that any other ready threads
	      // will run.   As such we revert yield_all() after some number of iterations.
	      // Yield_all() is implemented as a short unconditional sleep on some platforms.
	      // Typical operating systems round a "short" sleep period up to 10 msecs, so sleeping
	      // can actually increase the time it takes the VM thread to detect that a system-wide
	      // stop-the-world safepoint has been reached.  In a pathological scenario such as that
	      // described in CR6415670 the VMthread may sleep just before the mutator(s) become safe.
	      // In that case the mutators will be stalled waiting for the safepoint to complete and the
	      // the VMthread will be sleeping, waiting for the mutators to rendezvous.  The VMthread
	      // will eventually wake up and detect that all mutators are safe, at which point
	      // we'll again make progress.
	      //
	      // Beware too that that the VMThread typically runs at elevated priority.
	      // Its default priority is higher than the default mutator priority.
	      // Obviously, this complicates spinning.
	      //
	      // Note too that on Windows XP SwitchThreadTo() has quite different behavior than Sleep(0).
	      // Sleep(0) will _not yield to lower priority threads, while SwitchThreadTo() will.
	      //
	      // See the comments in synchronizer.cpp for additional remarks on spinning.
	      //
	      // In the future we might:
	      // 1. Modify the safepoint scheme to avoid potentally unbounded spinning.
	      //    This is tricky as the path used by a thread exiting the JVM (say on
	      //    on JNI call-out) simply stores into its state field.  The burden
	      //    is placed on the VM thread, which must poll (spin).
	      // 2. Find something useful to do while spinning.  If the safepoint is GC-related
	      //    we might aggressively scan the stacks of threads that are already safe.
	      // 3. Use Solaris schedctl to examine the state of the still-running mutators.
	      //    If all the mutators are ONPROC there's no reason to sleep or yield.
	      // 4. YieldTo() any still-running mutators that are ready but OFFPROC.
	      // 5. Check system saturation.  If the system is not fully saturated then
	      //    simply spin and avoid sleep/yield.
	      // 6. As still-running mutators rendezvous they could unpark the sleeping
	      //    VMthread.  This works well for still-running mutators that become
	      //    safe.  The VMthread must still poll for mutators that call-out.
	      // 7. Drive the policy on time-since-begin instead of iterations.
	      // 8. Consider making the spin duration a function of the # of CPUs:
	      //    Spin = (((ncpus-1) * M) + K) + F(still_running)
	      //    Alternately, instead of counting iterations of the outer loop
	      //    we could count the # of threads visited in the inner loop, above.
	      // 9. On windows consider using the return value from SwitchThreadTo()
	      //    to drive subsequent spin/SwitchThreadTo()/Sleep(N) decisions.
	
      {- -------------------------------------------
  (1.1.1) os::make_polling_page_unreadable() を呼んで, 
          Safepoint Polling page をアクセス不可にしておく.
          (DeferPollingPageLoopCount オプションが 0 以上の場合は, ここで行う.
           なお毎ループごとに行うわけではなく, ループ回数が DeferPollingPageLoopCount に
           達したとき(= iterations が DeferPollingPageLoopCount に等しくなったとき) に一度だけ行う.
           また, UseCompilerSafepoints オプションが指定されていない場合は, この処理は行わない.)
          ---------------------------------------- -}

	      if (UseCompilerSafepoints && int(iterations) == DeferPollingPageLoopCount) {
	         guarantee (PageArmed == 0, "invariant") ;
	         PageArmed = 1 ;
	         os::make_polling_page_unreadable();
	      }
	
      {- -------------------------------------------
  (1.1.1) 少しの間だけ待つ. 
          この処理は, 状況に合わせて以下のどれかを選択.
  
          * MP 環境(以下の ncpus が 1 より大)で, かつループ回数(以下の steps)が SafepointSpinBeforeYield 閾値未満の場合:
            SpinPause() で待機する
          * 上記以外で, ループ回数(以下の steps)が DeferThrSuspendLoopCount 閾値未満の場合:
            os::NakedYield() で待機する
          * それ以外の場合:
            os::yield_all() で待機する
    
          (なおコメントによると,
           "(ncpus > 1)" の代わりに, 
           "(still_running < (ncpus + EPSILON))" とか
           "((still_running + _waiting_to_block - TryingToBlock))" とかもいいかもしれない, 
           とのこと)
          ---------------------------------------- -}

	      // Instead of (ncpus > 1) consider either (still_running < (ncpus + EPSILON)) or
	      // ((still_running + _waiting_to_block - TryingToBlock)) < ncpus)
	      ++steps ;
	      if (ncpus > 1 && steps < SafepointSpinBeforeYield) {
	        SpinPause() ;     // MP-Polite spin
	      } else
	      if (steps < DeferThrSuspendLoopCount) {
	        os::NakedYield() ;
	      } else {
	        os::yield_all(steps) ;
	        // Alternately, the VM thread could transiently depress its scheduling priority or
	        // transiently increase the priority of the tardy mutator(s).
	      }
	
      {- -------------------------------------------
  (1.1.1) ループ回数を表す iterations 変数をインクリメント.
          ---------------------------------------- -}

	      iterations ++ ;
	    }

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    assert(iterations < (uint)max_jint, "We have been iterating in the safepoint loop too long");
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(still_running == 0, "sanity check");
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録)
      (See: SafepointSynchronize::update_statistics_on_spin_end())
      ---------------------------------------- -}

	  if (PrintSafepointStatistics) {
	    update_statistics_on_spin_end();
	  }
	
  {- -------------------------------------------
  (1) _waiting_to_block が 0 になるまで, 以下の while ループ内で待機する.
  
      (なお, _waiting_to_block を減少させる処理は, 以下の ２カ所で行われている.
       * この関数内の still_running を計算するループ内 (上述):
         あの時点で既に停止していたスレッドについては, VMThread が減少させている.
         See: ThreadSafepointState::examine_state_of_thread()
  
       * JavaThread が行う停止処理内:
         still_running が 0 になった後で停止したスレッドについては, 
         各 JavaThread 自身が減少させている.
         See: SafepointSynchronize::block()                            )
  
      待機処理は, Safepoint_lock に対して Monitior::wait() を繰り返すことで行う.
      (起こす処理は SafepointSynchronize::block() で行われている.
       See: SafepointSynchronize::block())
  
      SafepointTimeout オプションが指定されていなければ, 無期限の Monitior::wait() を行う.
      逆に, SafepointTimeout オプションが指定されている場合は, 
      最大で safepoint_limit_time 時間まで待機を行い, 
      タイムアウトしたら SafepointSynchronize::print_safepoint_timeout() でトレース出力を出している.
      (ただし, デバッグ用の DieOnSafepointTimeout オプションが指定されている場合は, 
       SafepointSynchronize::print_safepoint_timeout() を呼ぶと, 出力だけでなく強制終了処理も行われる)
  
  
      (ついでに, (トレース出力)も出している)
      ---------------------------------------- -}

	  // wait until all threads are stopped
	  while (_waiting_to_block > 0) {
	    if (TraceSafepoint) tty->print_cr("Waiting for %d thread(s) to block", _waiting_to_block);
	    if (!SafepointTimeout || timeout_error_printed) {
	      Safepoint_lock->wait(true);  // true, means with no safepoint checks
	    } else {
	      // Compute remaining time
	      jlong remaining_time = safepoint_limit_time - os::javaTimeNanos();
	
	      // If there is no remaining time, then there is an error
	      if (remaining_time < 0 || Safepoint_lock->wait(true, remaining_time / MICROUNITS)) {
	        print_safepoint_timeout(_blocking_timeout);
	      }
	    }
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_waiting_to_block == 0, "sanity check");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (SafepointTimeout) {
	    jlong current_time = os::javaTimeNanos();
	    if (safepoint_limit_time < current_time) {
	      tty->print_cr("# SafepointSynchronize: Finished after "
	                    INT64_FORMAT_W(6) " ms",
	                    ((current_time - safepoint_limit_time) / MICROUNITS +
	                     SafepointTimeoutDelay));
	    }
	  }
	#endif
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert((_safepoint_counter & 0x1) == 0, "must be even");
	  assert(Threads_lock->owned_by_self(), "must hold Threads_lock");

  {- -------------------------------------------
  (1) _safepoint_counter カウンタをインクリメントしておく.
  
      (これは JNI による高速フィールドアクセスのための処理.
       See: [here](no305911u.html) for details)
      ---------------------------------------- -}

	  _safepoint_counter ++;
	
  {- -------------------------------------------
  (1) SafepointSynchronize::_state を _synchronized に変更する.
      上に書かれているように順序も大事なので, OrderAccess::fence() でメモリバリアも張っている.
  
      (SafepointSynchronize::_state を変更したことで, 
       SafepointSynchronize::is_at_safepoint() が true を返すようになる.
       See: SafepointSynchronize::is_at_safepoint())
      ---------------------------------------- -}

	  // Record state
	  _state = _synchronized;
	
	  OrderAccess::fence();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceSafepoint) {
	    VM_Operation *op = VMThread::vm_operation();
	    tty->print_cr("Entering safepoint region: %s", (op != NULL) ? op->name() : "no vm operation");
	  }
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録)(JMM 用)  ("sun.rt.safepointSyncTime")
      (See: RuntimeService::record_safepoint_synchronized())
      ---------------------------------------- -}

	  RuntimeService::record_safepoint_synchronized();

  {- -------------------------------------------
  (1) (プロファイル情報の記録)
      (See: SafepointSynchronize::update_statistics_on_sync_end())
      ---------------------------------------- -}

	  if (PrintSafepointStatistics) {
	    update_statistics_on_sync_end(os::javaTimeNanos());
	  }
	
  {- -------------------------------------------
  (1) SafepointSynchronize::do_cleanup_tasks() を呼んで, 
      定期的に行う必要のある様々なクリーンアップ処理を実行しておく.
      ---------------------------------------- -}

	  // Call stuff that needs to be run when a safepoint is just about to be completed
	  do_cleanup_tasks();
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録)
      (See: SafepointSynchronize::update_statistics_on_cleanup_end())
      ---------------------------------------- -}

	  if (PrintSafepointStatistics) {
	    // Record how much time spend on the above cleanup tasks
	    update_statistics_on_cleanup_end(os::javaTimeNanos());
	  }
	  }
	}
	
```


