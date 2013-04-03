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
JvmtiEventControllerPrivate::env_dispose(JvmtiEnvBase *env) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads::number_of_threads() == 0 || JvmtiThreadState_lock->is_locked(), "sanity check");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [*] # env dispose"));
	
  {- -------------------------------------------
  (1) この JvmtiEnv に関する全てのイベント通知を(コールバック関数を NULL に設定することで)無効にしておく
      ---------------------------------------- -}

	  // Before the environment is marked disposed, disable all events on this
	  // environment (by zapping the callbacks).  As a result, the disposed
	  // environment will not call event handlers.
	  set_event_callbacks(env, NULL, 0);
	  for (jint extension_event_index = EXT_MIN_EVENT_TYPE_VAL;
	       extension_event_index <= EXT_MAX_EVENT_TYPE_VAL;
	       ++extension_event_index) {
	    set_extension_event_callback(env, extension_event_index, NULL);
	  }
	
  {- -------------------------------------------
  (1) env 引数に対して, JvmtiEnvBase::env_dispose() を呼び出す.
      ---------------------------------------- -}

	  // Let the environment finish disposing itself.
	  env->env_dispose();
	}
	
```


