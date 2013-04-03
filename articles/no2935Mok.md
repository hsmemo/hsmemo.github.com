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
void JvmtiEventControllerPrivate::set_event_callbacks(JvmtiEnvBase *env,
                                                      const jvmtiEventCallbacks* callbacks,
                                                      jint size_of_callbacks) {
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

	  EC_TRACE(("JVMTI [*] # set event callbacks"));
	
  {- -------------------------------------------
  (1) JvmtiEnvBase::set_event_callbacks() を呼んで, callbacks 引数で指定されたコールバック関数を登録する.
      ---------------------------------------- -}

	  env->set_event_callbacks(callbacks, size_of_callbacks);

  {- -------------------------------------------
  (1) 現在登録されているコールバック関数を全て調査し, それら全部の和集合を表す bit mask を作って
      env->env_event_enable()->_event_callback_enabled() にセットする.
  
      (これは, 「env 引数で指定された JvmtiEnvBase オブジェクト中の JvmtiEnvEventEnable オブジェクトの
       _event_callback_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」, ということ.)
      ---------------------------------------- -}

	  jlong enabled_bits = 0;
	  for (int ei = JVMTI_MIN_EVENT_TYPE_VAL; ei <= JVMTI_MAX_EVENT_TYPE_VAL; ++ei) {
	    jvmtiEvent evt_t = (jvmtiEvent)ei;
	    if (env->has_callback(evt_t)) {
	      enabled_bits |= JvmtiEventEnabled::bit_for(evt_t);
	    }
	  }
	  env->env_event_enable()->_event_callback_enabled.set_bits(enabled_bits);

  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::recompute_enabled() を呼んで, 
      "truly enabled event" 情報を更新する.
      ---------------------------------------- -}

	  recompute_enabled();
	}
	
```


