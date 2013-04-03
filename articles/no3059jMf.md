---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp

### 名前(function name)
```
void
JvmtiEventControllerPrivate::thread_ended(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Removes the JvmtiThreadState associated with the specified thread.
	  // May be called after all environments have been disposed.
	  assert(JvmtiThreadState_lock->is_locked(), "sanity check");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [%s] # thread ended", JvmtiTrace::safe_get_thread_name(thread)));
	
  {- -------------------------------------------
  (1) 引数で指定されたスレッドに対応する JvmtiThreadState を開放.
      ---------------------------------------- -}

	  JvmtiThreadState *state = thread->jvmti_thread_state();
	  assert(state != NULL, "else why are we here?");
	  delete state;
	}
	
```


