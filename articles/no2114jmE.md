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
bool os::create_thread(Thread* thread, ThreadType thr_type, size_t stack_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) OSThread オブジェクトを生成する (失敗したらここでリターン)
      ---------------------------------------- -}

	  // Allocate the OSThread object
	  OSThread* osthread = new OSThread(NULL, NULL);
	  if (osthread == NULL) {
	    return false;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if ( ThreadPriorityVerbose ) {
	    char *thrtyp;
	    switch ( thr_type ) {
	      case vm_thread:
	        thrtyp = (char *)"vm";
	        break;
	      case cgc_thread:
	        thrtyp = (char *)"cgc";
	        break;
	      case pgc_thread:
	        thrtyp = (char *)"pgc";
	        break;
	      case java_thread:
	        thrtyp = (char *)"java";
	        break;
	      case compiler_thread:
	        thrtyp = (char *)"compiler";
	        break;
	      case watcher_thread:
	        thrtyp = (char *)"watcher";
	        break;
	      default:
	        thrtyp = (char *)"unknown";
	        break;
	    }
	    tty->print_cr("In create_thread, creating a %s thread\n", thrtyp);
	  }
	
  {- -------------------------------------------
  (1) スレッドに設定するスタックサイズを求める
      (ただし, 引数でスタックサイズが指定されている場合 (引数の stack_size が 0 ではない場合) には
       それを使うだけなので, 以下の処理は行わない)
  
      具体的には, スレッド種別に応じた以下値をスタックサイズとする.
      (ただし, 以下の値が os::Solaris::min_stack_allowed より小さい場合は, 
       os::Solaris::min_stack_allowed とする)
      * JavaThread の場合
        JavaThread::stack_size_at_create() の値を使用
        (これは -Xss オプションで指定した値.
         See: Arguments::parse_each_vm_init_arg(), os::init_2())
      * CompilerThread の場合
        CompilerThreadStackSize オプションの値を使用.
        (ただし, CompilerThreadStackSize オプションが指定されていない場合(値が 0 の場合)には, 
         VMThread 等と同じ方法で決める)
      * VMThread, Parallel GC thread, Concurrent GC thread, WatcherThread の場合
        VMThreadStackSize オプションの値を使用.
        (ただし, VMThreadStackSize オプションが指定されていない場合(値が 0 の場合)には, 
         1M (LP64 なら 2M) とする)
      ---------------------------------------- -}

	  // Calculate stack size if it's not specified by caller.
	  if (stack_size == 0) {
	    // The default stack size 1M (2M for LP64).
	    stack_size = (BytesPerWord >> 2) * K * K;
	
	    switch (thr_type) {
	    case os::java_thread:
	      // Java threads use ThreadStackSize which default value can be changed with the flag -Xss
	      if (JavaThread::stack_size_at_create() > 0) stack_size = JavaThread::stack_size_at_create();
	      break;
	    case os::compiler_thread:
	      if (CompilerThreadStackSize > 0) {
	        stack_size = (size_t)(CompilerThreadStackSize * K);
	        break;
	      } // else fall through:
	        // use VMThreadStackSize if CompilerThreadStackSize is not defined
	    case os::vm_thread:
	    case os::pgc_thread:
	    case os::cgc_thread:
	    case os::watcher_thread:
	      if (VMThreadStackSize > 0) stack_size = (size_t)(VMThreadStackSize * K);
	      break;
	    }
	  }
	  stack_size = MAX2(stack_size, os::Solaris::min_stack_allowed);
	
  {- -------------------------------------------
  (1) 生成したスレッドの ThreadState を ALLOCATED に変更する.
      ---------------------------------------- -}

	  // Initial state is ALLOCATED but not INITIALIZED
	  osthread->set_state(ALLOCATED);
	
  {- -------------------------------------------
  (1) 生成したスレッド数が os::Solaris::_os_thread_limit を超えている場合には, 
      まだメモリ空間に余裕があるかどうかを確認しておく.
      駄目ならここでリターン.
      ---------------------------------------- -}

	  if (os::Solaris::_os_thread_count > os::Solaris::_os_thread_limit) {
	    // We got lots of threads. Check if we still have some address space left.
	    // Need to be at least 5Mb of unreserved address space. We do check by
	    // trying to reserve some.
	    const size_t VirtualMemoryBangSize = 20*K*K;
	    char* mem = os::reserve_memory(VirtualMemoryBangSize);
	    if (mem == NULL) {
	      delete osthread;
	      return false;
	    } else {
	      // Release the memory again
	      os::release_memory(mem, VirtualMemoryBangSize);
	    }
	  }
	
  {- -------------------------------------------
  (1) Thread::set_osthread() を呼んで, 生成した OSThread を
      引数で渡された Thread オブジェクトに対応づける
      ---------------------------------------- -}

	  // Setup osthread because the child thread may need it.
	  thread->set_osthread(osthread);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (flags は, thr_create() する際の引数.
       THR_SUSPENDED は常に付ける. 
       また UseDetachedThreads オプションが指定されている場合には THR_DETACHED も指定し, 
       LWP と 1対1対応させたいケースでは THR_BOUND も指定する.)
      ---------------------------------------- -}

	  // Create the Solaris thread
	  // explicit THR_BOUND for T2_libthread case in case
	  // that assumption is not accurate, but our alternate signal stack
	  // handling is based on it which must have bound threads
	  thread_t tid = 0;
	  long     flags = (UseDetachedThreads ? THR_DETACHED : 0) | THR_SUSPENDED
	                   | ((UseBoundThreads || os::Solaris::T2_libthread() ||
	                       (thr_type == vm_thread) ||
	                       (thr_type == cgc_thread) ||
	                       (thr_type == pgc_thread) ||
	                       (thr_type == compiler_thread && BackgroundCompilation)) ?
	                      THR_BOUND : 0);
	  int      status;
	
  {- -------------------------------------------
  (1) 物理コア数に対して十分な LWP が割り当てられないことがあるので, 
      必要に応じて thr_setconcurrency() を呼んで少し小細工をしておく.
  
      (コメントによると, 30-way のシステムで 2~3 LWP しか割り当てられないこともあるとか.
       なお, 以下のコードは対処療法であって, THR_BOUND で LWP と対応づける等の方が正しい, とのこと)
      ---------------------------------------- -}

	  // 4376845 -- libthread/kernel don't provide enough LWPs to utilize all CPUs.
	  //
	  // On multiprocessors systems, libthread sometimes under-provisions our
	  // process with LWPs.  On a 30-way systems, for instance, we could have
	  // 50 user-level threads in ready state and only 2 or 3 LWPs assigned
	  // to our process.  This can result in under utilization of PEs.
	  // I suspect the problem is related to libthread's LWP
	  // pool management and to the kernel's SIGBLOCKING "last LWP parked"
	  // upcall policy.
	  //
	  // The following code is palliative -- it attempts to ensure that our
	  // process has sufficient LWPs to take advantage of multiple PEs.
	  // Proper long-term cures include using user-level threads bound to LWPs
	  // (THR_BOUND) or using LWP-based synchronization.  Note that there is a
	  // slight timing window with respect to sampling _os_thread_count, but
	  // the race is benign.  Also, we should periodically recompute
	  // _processors_online as the min of SC_NPROCESSORS_ONLN and the
	  // the number of PEs in our partition.  You might be tempted to use
	  // THR_NEW_LWP here, but I'd recommend against it as that could
	  // result in undesirable growth of the libthread's LWP pool.
	  // The fix below isn't sufficient; for instance, it doesn't take into count
	  // LWPs parked on IO.  It does, however, help certain CPU-bound benchmarks.
	  //
	  // Some pathologies this scheme doesn't handle:
	  // *  Threads can block, releasing the LWPs.  The LWPs can age out.
	  //    When a large number of threads become ready again there aren't
	  //    enough LWPs available to service them.  This can occur when the
	  //    number of ready threads oscillates.
	  // *  LWPs/Threads park on IO, thus taking the LWP out of circulation.
	  //
	  // Finally, we should call thr_setconcurrency() periodically to refresh
	  // the LWP pool and thwart the LWP age-out mechanism.
	  // The "+3" term provides a little slop -- we want to slightly overprovision.
	
	  if (AdjustConcurrency && os::Solaris::_os_thread_count < (_processors_online+3)) {
	    if (!(flags & THR_BOUND)) {
	      thr_setconcurrency (os::Solaris::_os_thread_count);       // avoid starvation
	    }
	  }

  {- -------------------------------------------
  (1) (デバッグ用の処理) (DEBUG_ONLY 時にのみ実行)
      ---------------------------------------- -}

	  // Although this doesn't hurt, we should warn of undefined behavior
	  // when using unbound T1 threads with schedctl().  This should never
	  // happen, as the compiler and VM threads are always created bound
	  DEBUG_ONLY(
	      if ((VMThreadHintNoPreempt || CompilerThreadHintNoPreempt) &&
	          (!os::Solaris::T2_libthread() && (!(flags & THR_BOUND))) &&
	          ((thr_type == vm_thread) || (thr_type == cgc_thread) ||
	           (thr_type == pgc_thread) || (thr_type == compiler_thread && BackgroundCompilation))) {
	         warning("schedctl behavior undefined when Compiler/VM/GC Threads are Unbound");
	      }
	  );
	
	
  {- -------------------------------------------
  (1) lwp_id フィールドや thread_id フィールドの値を初期化しておく.
      (ここでは -1 としておく. 
       これにより, スレッドが開始される前に優先度をうっかりセットしてしまうのを防ぐ)
      ---------------------------------------- -}

	  // Mark that we don't have an lwp or thread id yet.
	  // In case we attempt to set the priority before the thread starts.
	  osthread->set_lwp_id(-1);
	  osthread->set_thread_id(-1);
	
  {- -------------------------------------------
  (1) thr_create() でスレッドを生成する.
      (なお, スレッドのエントリポイントは java_start() としている)
      (なお, flags で THR_SUSPENDED を指定しているため, 
       実際に動き始めるのは os::pd_start_thread() で thr_continue() されてから.
       See: os::pd_start_thread())
  
      (スレッド生成に失敗した場合は, 
      生成したりソースを解放し, ここでリターン)
      ---------------------------------------- -}

	  status = thr_create(NULL, stack_size, java_start, thread, flags, &tid);
	  if (status != 0) {
	    if (PrintMiscellaneous && (Verbose || WizardMode)) {
	      perror("os::create_thread");
	    }
	    thread->set_osthread(NULL);
	    // Need to clean up stuff we've allocated so far
	    delete osthread;
	    return false;
	  }
	
  {- -------------------------------------------
  (1) os::Solaris::_os_thread_count フィールドの値を増加させておく.
      (この値は, os::create_thread() 内でのランタイムチェックや調整処理に使われている.
       See: os::create_thread())
      ---------------------------------------- -}

	  Atomic::inc(&os::Solaris::_os_thread_count);
	
  {- -------------------------------------------
  (1) OSThread::set_thread_id() を呼んで, 生成したスレッドの thread_t を 
      OSThread に対応づける
      ---------------------------------------- -}

	  // Store info on the Solaris thread into the OSThread
	  osthread->set_thread_id(tid);
	
  {- -------------------------------------------
  (1) (HotSpot 内で生成したスレッドなので)
      OSThread::set_vm_created() を呼んでそのことを記録しておく.
      ---------------------------------------- -}

	  // Remember that we created this thread so we can set priority on it
	  osthread->set_vm_created();
	
  {- -------------------------------------------
  (1) thr_setprio() でスレッドの優先度の設定をしておく.
      (なお, UseThreadPriorities オプションが指定されていない場合は, 何もしない)
      ---------------------------------------- -}

	  // Set the default thread priority otherwise use NormalPriority
	
	  if ( UseThreadPriorities ) {
	     thr_setprio(tid, (DefaultThreadPriority == -1) ?
	                        java_to_os_priority[NormPriority] :
	                        DefaultThreadPriority);
	  }
	
  {- -------------------------------------------
  (1) 生成したスレッドの ThreadState を INITIALIZED に変更する.
      ---------------------------------------- -}

	  // Initial thread state is INITIALIZED, not SUSPENDED
	  osthread->set_state(INITIALIZED);
	
  {- -------------------------------------------
  (1) リターン
      (この段階では, 生成したスレッドは INITIALIZED 状態で有り, 稼働はしていない(suspend されている状態).
       スレッドの開始処理は, ここからリターンした後の関数によって行われる.)
      ---------------------------------------- -}

	  // The thread is returned suspended (in state INITIALIZED), and is started higher up in the call chain
	  return true;
	}
	
```


