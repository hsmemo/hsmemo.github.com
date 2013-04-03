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
JvmtiEventController::env_dispose(JvmtiEnvBase *env) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::env_dispose() を呼び出す.
    
      (なお, HotSpot の起動時(= Threads::number_of_threads() が 0)であれば
       ロックは必要ない(というかロック機構自体が立ち上がっていない)のでロックを取らずに呼び出す.
       そうでなければ JvmtiThreadState_lock を確保した状態で呼び出す.)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {
	    // during early VM start-up locks don't exist, but we are safely single threaded,
	    // call the functionality without holding the JvmtiThreadState_lock.
	    JvmtiEventControllerPrivate::env_dispose(env);
	  } else {
	    MutexLocker mu(JvmtiThreadState_lock);
	    JvmtiEventControllerPrivate::env_dispose(env);
	  }
	}
	
```


