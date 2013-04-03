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
JvmtiEventController::set_extension_event_callback(JvmtiEnvBase *env,
                                                   jint extension_event_index,
                                                   jvmtiExtensionEvent callback) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::set_extension_event_callback() を呼び出す.
  
      (なお, HotSpot の起動時(= Threads::number_of_threads() が 0)であれば
       ロックは必要ない(というかロック機構自体が立ち上がっていない)のでロックを取らずに呼び出す.
       そうでなければ JvmtiThreadState_lock を確保した状態で呼び出す.)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {
	    JvmtiEventControllerPrivate::set_extension_event_callback(env, extension_event_index, callback);
	  } else {
	    MutexLocker mu(JvmtiThreadState_lock);
	    JvmtiEventControllerPrivate::set_extension_event_callback(env, extension_event_index, callback);
	  }
	}
	
```


