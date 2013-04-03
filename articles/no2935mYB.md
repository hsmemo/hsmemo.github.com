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
JvmtiEventController::set_user_enabled(JvmtiEnvBase *env, JavaThread *thread, jvmtiEvent event_type, bool enabled) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::set_user_enabled() を呼び出す.
  
      (なお, HotSpot の起動中(= Threads::number_of_threads() が 0)であれば
       (他にスレッドがいないので) この呼び出し時に排他処理は必要ない.
       起動中でなければ, JvmtiThreadState_lock で排他した状態で呼び出す.)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {
	    // during early VM start-up locks don't exist, but we are safely single threaded,
	    // call the functionality without holding the JvmtiThreadState_lock.
	    JvmtiEventControllerPrivate::set_user_enabled(env, thread, event_type, enabled);
	  } else {
	    MutexLocker mu(JvmtiThreadState_lock);
	    JvmtiEventControllerPrivate::set_user_enabled(env, thread, event_type, enabled);
	  }
	}
	
```


