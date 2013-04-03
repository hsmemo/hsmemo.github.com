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
JvmtiEventControllerPrivate::set_user_enabled(JvmtiEnvBase *env, JavaThread *thread,
                                          jvmtiEvent event_type, bool enabled) {
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

	  EC_TRACE(("JVMTI [%s] # user %s event %s",
	            thread==NULL? "ALL": JvmtiTrace::safe_get_thread_name(thread),
	            enabled? "enabled" : "disabled", JvmtiTrace::event_name(event_type)));
	
  {- -------------------------------------------
  (1) JvmtiEventEnabled を更新する. この処理は引数に応じて２通り.
    
      * 処理対象のスレッドが指定されていない場合 (= thread 引数が NULL の場合): 
    
        JvmtiEnvEventEnable::set_user_enabled() を呼んで, 
        「env 引数で指定された JvmtiEnvBase オブジェクト中の JvmtiEnvEventEnable オブジェクトの
         _event_user_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」を更新する.
  
      * 〃指定されている場合: 
  
        JvmtiEnvThreadEventEnable::set_user_enabled() を呼んで, 
        「thread 引数で指定された JavaThread に対応する JvmtiThreadState オブジェクト中の 
         JvmtiEnvThreadState オブジェクト中の JvmtiEnvThreadEventEnable オブジェクト中の
         _event_user_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」を更新する.
      ---------------------------------------- -}

	  if (thread == NULL) {
	    env->env_event_enable()->set_user_enabled(event_type, enabled);
	  } else {
	    // create the thread state (if it didn't exist before)
	    JvmtiThreadState *state = JvmtiThreadState::state_for_while_locked(thread);
	    if (state != NULL) {
	      state->env_thread_state(env)->event_enable()->set_user_enabled(event_type, enabled);
	    }
	  }

  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::recompute_enabled() を呼んで, 
      "truly enabled event" 情報を更新する.
      ---------------------------------------- -}

	  recompute_enabled();
	}
	
```


