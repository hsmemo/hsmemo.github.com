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
JvmtiEnv::RawMonitorWait(JvmtiRawMonitor * rmonitor, jlong millis) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int r;
	  Thread* thread = Thread::current();
	
  {- -------------------------------------------
  (1) (以下の処理は, カレントスレッドが JavaThread かそれ以外(VMThread or ConcurrentGCThread)かで2通りに分かれる)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は, カレントスレッドが JavaThread の場合)
      ---------------------------------------- -}

	  if (thread->is_Java_thread()) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    JavaThread* current_thread = (JavaThread*)thread;

    {- -------------------------------------------
  (1.1) JvmtiRawMonitor::raw_wait() を呼んで待機する.
    
        (なんだか JavaThreadState の変更処理で手こずっているようだが... #TODO)
        ---------------------------------------- -}

	#ifdef PROPER_TRANSITIONS
	    // Not really unknown but ThreadInVMfromNative does more than we want
	    ThreadInVMfromUnknown __tiv;
	    {
	      ThreadBlockInVM __tbivm(current_thread);
	      r = rmonitor->raw_wait(millis, true, current_thread);
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
	
	    r = rmonitor->raw_wait(millis, true, current_thread);
	    // restore state, still at a safepoint safe state
	    current_thread->set_thread_state(state);
	
	#endif /* PROPER_TRANSITIONS */

  {- -------------------------------------------
  (1) (以下は, カレントスレッドが VMThread or ConcurrentGCThread の場合)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) JvmtiRawMonitor::raw_wait() を呼んで待機する.
        ---------------------------------------- -}

	    if (thread->is_VM_thread() || thread->is_ConcurrentGC_thread()) {
	      r = rmonitor->raw_wait(millis, true, thread);
	    } else {
	      ShouldNotReachHere();
	    }
	  }
	
  {- -------------------------------------------
  (1) リターンする.
      
      なお, JvmtiRawMonitor::raw_wait() が失敗していた場合は, 適切なエラーをリターンする.
      * 返値が ObjectMonitor::OM_INTERRUPTED の場合:
        JVMTI_ERROR_INTERRUPT をリターン
      * 返値が ObjectMonitor::OM_ILLEGAL_MONITOR_STATE の場合:
        JVMTI_ERROR_NOT_MONITOR_OWNER をリターン
      * 返値がその他の ObjectMonitor::OM_OK ではない値の場合: (このパスはあり得なさそうだが...)
        JVMTI_ERROR_INTERNAL をリターン
      ---------------------------------------- -}

	  switch (r) {
	  case ObjectMonitor::OM_INTERRUPTED:
	    return JVMTI_ERROR_INTERRUPT;
	  case ObjectMonitor::OM_ILLEGAL_MONITOR_STATE:
	    return JVMTI_ERROR_NOT_MONITOR_OWNER;
	  }
	  assert(r == ObjectMonitor::OM_OK, "raw_wait should have worked");
	  if (r != ObjectMonitor::OM_OK) {  // robustness
	    return JVMTI_ERROR_INTERNAL;
	  }
	
	  return JVMTI_ERROR_NONE;
	} /* end RawMonitorWait */
	
```


