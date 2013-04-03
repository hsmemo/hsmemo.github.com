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
JvmtiEventController::thread_ended(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::thread_ended() を呼び出すだけ.
      ---------------------------------------- -}

	  // operates only on the current thread
	  // JvmtiThreadState_lock grabbed only if needed.
	  JvmtiEventControllerPrivate::thread_ended(thread);
	}
	
```


