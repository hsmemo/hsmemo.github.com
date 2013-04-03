---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// rmonitor - pre-checked for validity
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::RawMonitorEnter(JvmtiRawMonitor * rmonitor) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は, HotSpot の起動中(= Threads::number_of_threads() が 0)かどうかで ２通りに分かれる)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は HotSpot の起動中(= Threads::number_of_threads() が 0)の場合)
      (この段階では JavaThreads オブジェクトが作れていないので ObjectMonitor も使えない)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {

    {- -------------------------------------------
  (1.1) JvmtiPendingMonitors::enter() を呼んで, JvmtiPendingMonitors 内の pending list に追加する.
        ---------------------------------------- -}

	    // No JavaThreads exist so ObjectMonitor enter cannot be
	    // used, add this raw monitor to the pending list.
	    // The pending monitors will be actually entered when
	    // the VM is setup.
	    // See transition_pending_raw_monitors in create_vm()
	    // in thread.cpp.
	    JvmtiPendingMonitors::enter(rmonitor);

  {- -------------------------------------------
  (1) (以下は HotSpot の起動完了後の場合)
      (この場合の処理は, カレントスレッドが JavaThread かそれ以外(VMThread or ConcurrentGCThread)かで2通りに分かれる)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    int r;
	    Thread* thread = Thread::current();
	
    {- -------------------------------------------
  (1.1) (以下は, カレントスレッドが JavaThread の場合)
        ---------------------------------------- -}

	    if (thread->is_Java_thread()) {

      {- -------------------------------------------
  (1.1.1) (変数宣言など)
          ---------------------------------------- -}

	      JavaThread* current_thread = (JavaThread*)thread;
	
      {- -------------------------------------------
  (1.1.1) JvmtiRawMonitor::raw_enter() を呼んでロックを取得する.
    
          (なんだか JavaThreadState の変更処理で手こずっているようだが... #TODO)
          ---------------------------------------- -}

	#ifdef PROPER_TRANSITIONS
	      // Not really unknown but ThreadInVMfromNative does more than we want
	      ThreadInVMfromUnknown __tiv;
	      {
	        ThreadBlockInVM __tbivm(current_thread);
	        r = rmonitor->raw_enter(current_thread);
	      }
	#else
	      /* Transition to thread_blocked without entering vm state          */
	      /* This is really evil. Normally you can't undo _thread_blocked    */
	      /* transitions like this because it would cause us to miss a       */
	      /* safepoint but since the thread was already in _thread_in_native */
	      /* the thread is not leaving a safepoint safe state and it will    */
	      /* block when it tries to return from native. We can't safepoint   */
	      /* block in here because we could deadlock the vmthread. Blech.    */
	
	      JavaThreadState state = current_thread->thread_state();
	      assert(state == _thread_in_native, "Must be _thread_in_native");
	      // frame should already be walkable since we are in native
	      assert(!current_thread->has_last_Java_frame() ||
	             current_thread->frame_anchor()->walkable(), "Must be walkable");
	      current_thread->set_thread_state(_thread_blocked);
	
	      r = rmonitor->raw_enter(current_thread);
	      // restore state, still at a safepoint safe state
	      current_thread->set_thread_state(state);
	
	#endif /* PROPER_TRANSITIONS */

      {- -------------------------------------------
  (1.1.1) (assert)
          ---------------------------------------- -}

	      assert(r == ObjectMonitor::OM_OK, "raw_enter should have worked");

    {- -------------------------------------------
  (1.1) (以下は, カレントスレッドが VMThread or ConcurrentGCThread の場合)
        ---------------------------------------- -}

	    } else {

      {- -------------------------------------------
  (1.1.1) JvmtiRawMonitor::raw_enter() を呼んでロックを取得する.
          ---------------------------------------- -}

	      if (thread->is_VM_thread() || thread->is_ConcurrentGC_thread()) {
	        r = rmonitor->raw_enter(thread);
	      } else {
	        ShouldNotReachHere();
	      }
	    }
	
    {- -------------------------------------------
  (1.1) もし JvmtiRawMonitor::raw_enter() が失敗していたら, JVMTI_ERROR_INTERNAL をリターン. (このパスはあり得なさそうだが...)
        ---------------------------------------- -}

	    if (r != ObjectMonitor::OM_OK) {  // robustness
	      return JVMTI_ERROR_INTERNAL;
	    }
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end RawMonitorEnter */
	
```


