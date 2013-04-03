---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// Threads::destroy_vm() is normally called from jni_DestroyJavaVM() when
// the program falls off the end of main(). Another VM exit path is through
// vm_exit() when the program calls System.exit() to return a value or when
// there is a serious error in VM. The two shutdown paths are not exactly
// the same, but they share Shutdown.shutdown() at Java level and before_exit()
// and VM_Exit op at VM level.
//
// Shutdown sequence:
//   + Wait until we are the last non-daemon thread to execute
//     <-- every thing is still working at this moment -->
//   + Call java.lang.Shutdown.shutdown(), which will invoke Java level
//        shutdown hooks, run finalizers if finalization-on-exit
//   + Call before_exit(), prepare for VM exit
//      > run VM level shutdown hooks (they are registered through JVM_OnExit(),
//        currently the only user of this mechanism is File.deleteOnExit())
//      > stop flat profiler, StatSampler, watcher thread, CMS threads,
//        post thread end and vm death events to JVMTI,
//        stop signal thread
//   + Call JavaThread::exit(), it will:
//      > release JNI handle blocks, remove stack guard pages
//      > remove this thread from Threads list
//     <-- no more Java code from this thread after this point -->
//   + Stop VM thread, it will bring the remaining VM to a safepoint and stop
//     the compiler threads at safepoint
//     <-- do not use anything that could get blocked by Safepoint -->
//   + Disable tracing at JNI/JVM barriers
//   + Set _vm_exited flag for threads that are still running native code
//   + Delete this thread
//   + Call exit_globals()
//      > deletes tty
//      > deletes PerfMemory resources
//   + Return to caller

```

### 名前(function name)
```
bool Threads::destroy_vm() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* thread = JavaThread::current();
	
  {- -------------------------------------------
  (1) 現在のスレッドが唯一のユーザスレッド(non-daemon thread to execute)になるまで待つ.
    
      (ここでは Threads_lock に対して wait() することで待機.
       スレッドの終了処理時に, もし残りスレッドが 1 になっていれば, Threads_lock->notify_all() が呼ばれる.
       See: Threads::remove())
      ---------------------------------------- -}

	  // Wait until we are the last non-daemon thread to execute
	  { MutexLocker nu(Threads_lock);
	    while (Threads::number_of_non_daemon_threads() > 1 )
	      // This wait should make safepoint checks, wait without a timeout,
	      // and wait as a suspend-equivalent condition.
	      //
	      // Note: If the FlatProfiler is running and this thread is waiting
	      // for another non-daemon thread to finish, then the FlatProfiler
	      // is waiting for the external suspend request on this thread to
	      // complete. wait_for_ext_suspend_completion() will eventually
	      // timeout, but that takes time. Making this wait a suspend-
	      // equivalent condition solves that timeout problem.
	      //
	      Threads_lock->wait(!Mutex::_no_safepoint_check_flag, 0,
	                         Mutex::_as_suspend_equivalent_flag);
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Hang forever on exit if we are reporting an error.
	  if (ShowMessageBoxOnError && is_error_reported()) {
	    os::infinite_sleep();
	  }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  os::wait_for_keypress_at_exit();
	
  {- -------------------------------------------
  (1) JDK バージョンが 1.2 の場合は Universe::run_finalizers_on_exit() で finalizer を実行させる.
      ---------------------------------------- -}

	  if (JDK_Version::is_jdk12x_version()) {
	    // We are the last thread running, so check if finalizers should be run.
	    // For 1.3 or later this is done in thread->invoke_shutdown_hooks()
	    HandleMark rm(thread);
	    Universe::run_finalizers_on_exit();

  {- -------------------------------------------
  (1) JDK バージョンが 1.2 でない場合は, JavaThread::invoke_shutdown_hooks() で 
      java.lang.Shutdown.shutdown() を呼び出し, shutdown 用のフック関数やfinalizer処理を行う.
      ---------------------------------------- -}

	  } else {
	    // run Java level shutdown hooks
	    thread->invoke_shutdown_hooks();
	  }
	
  {- -------------------------------------------
  (1) before_exit() を呼び出し, 以下の処理を行う.
  
      * JVM_OnExit() で登録されたフック関数への準備をする   (<= ただし, JVM_OnExit() は使われていないような...).
      * Profiler, StatSampler, Watcher, GC threads を停止させる.
      * JVMTI/PI に static event を投げて, JVMPI を停止させ, Signal スレッドを停止させる.
      ---------------------------------------- -}

	  before_exit(thread);
	
  {- -------------------------------------------
  (1) JavaThread::exit() を呼び出し, 以下の処理を行う.
  
      * JNI handle block を解放
      * stack guard pages を解放
      * このスレッドを Threads list から削除する.
  
      (この段階で HotSpot は Java のコードが実行不可能な状態になる)
      ---------------------------------------- -}

	  thread->exit(true);
	
  {- -------------------------------------------
  (1) VMThread を停止させる.
  
      (これにより VM は safepoint 状態になり, Compiler スレッドも停止される)
      (safepoint では Safepoint によってブロックされてしまう処理を行わないよう気をつけないといけない(??))
      ---------------------------------------- -}

	  // Stop VM thread.
	  {
	    // 4945125 The vm thread comes to a safepoint during exit.
	    // GC vm_operations can get caught at the safepoint, and the
	    // heap is unparseable if they are caught. Grab the Heap_lock
	    // to prevent this. The GC vm_operations will not be able to
	    // queue until after the vm thread is dead.
	    MutexLocker ml(Heap_lock);
	
	    VMThread::wait_for_vm_thread_exit();
	    assert(SafepointSynchronize::is_at_safepoint(), "VM thread should exit at Safepoint");
	    VMThread::destroy();
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (!defined(PRODUCT) 時にのみ実行)
  
      IdealGraphPrinter を終了させる (See: IdealGraphPrinter).
      ---------------------------------------- -}

	  // clean up ideal graph printers
	#if defined(COMPILER2) && !defined(PRODUCT)
	  IdealGraphPrinter::clean_up();
	#endif
	
  {- -------------------------------------------
  (1) (この時点で daemon thread 以外の Java スレッドは停止している. 
       また daemon thread でも native コードを実行していないものは safeoint で停止している.
  
       ただし, native コードを実行している daemon thread はまだ動いている.
       こいつらを kill したり suspend させたりすると, デッドロックしかねないので, それはしてはいけない)
      ---------------------------------------- -}

	  // Now, all Java threads are gone except daemon threads. Daemon threads
	  // running Java code or in VM are stopped by the Safepoint. However,
	  // daemon threads executing native code are still running.  But they
	  // will be stopped at native=>Java/VM barriers. Note that we can't
	  // simply kill or suspend them, as it is inherently deadlock-prone.
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifndef PRODUCT 時にのみ実行)
  
      tracing at JNI/JVM barriers の停止
      ---------------------------------------- -}

	#ifndef PRODUCT
	  // disable function tracing at JNI/JVM barriers
	  TraceJNICalls = false;
	  TraceJVMCalls = false;
	  TraceRuntimeCalls = false;
	#endif
	
  {- -------------------------------------------
  (1) まだネイティブコードを実行中のスレッドのために
      _vm_exited フラグをセットして終了したと通達する
      ---------------------------------------- -}

	  VM_Exit::set_vm_exited();
	
  {- -------------------------------------------
  (1) VM が終了することを notify_vm_shutdown() で通知する.
      (なお, 現在は DTrace のフック点があるだけ)
      ---------------------------------------- -}

	  notify_vm_shutdown();
	
  {- -------------------------------------------
  (1) このスレッドを削除する
      ---------------------------------------- -}

	  delete thread;
	
  {- -------------------------------------------
  (1) exit_globals() で IO や PerfMonitor 用のリソースを解放する
      ---------------------------------------- -}

	  // exit_globals() will delete tty
	  exit_globals();
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return true;
	}
	
```


