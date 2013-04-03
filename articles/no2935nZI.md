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
JvmtiEventControllerPrivate::set_extension_event_callback(JvmtiEnvBase *env,
                                                          jint extension_event_index,
                                                          jvmtiExtensionEvent callback)
{
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

	  EC_TRACE(("JVMTI [*] # set extension event callback"));
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // extension events are allocated below JVMTI_MIN_EVENT_TYPE_VAL
	  assert(extension_event_index >= (jint)EXT_MIN_EVENT_TYPE_VAL &&
	         extension_event_index <= (jint)EXT_MAX_EVENT_TYPE_VAL, "sanity check");
	
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // As the bits for both standard (jvmtiEvent) and extension
	  // (jvmtiExtEvents) are stored in the same word we cast here to
	  // jvmtiEvent to set/clear the bit for this extension event.
	  jvmtiEvent event_type = (jvmtiEvent)extension_event_index;
	
	  // Prevent a possible race condition where events are re-enabled by a call to
	  // set event callbacks, where the DisposeEnvironment occurs after the boiler-plate
	  // environment check and before the lock is acquired.
	  // We can safely do the is_valid check now, as JvmtiThreadState_lock is held.
	  bool enabling = (callback != NULL) && (env->is_valid());
	  env->env_event_enable()->set_user_enabled(event_type, enabling);
	
	  // update the callback
	  jvmtiExtEventCallbacks* ext_callbacks = env->ext_callbacks();
	  switch (extension_event_index) {
	    case EXT_EVENT_CLASS_UNLOAD :
	      ext_callbacks->ClassUnload = callback;
	      break;
	    default:
	      ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) env->env_event_enable()->_event_callback_enabled の値を以下のように変更する.
      (これは, 「env 引数で指定された JvmtiEnvBase オブジェクト中の JvmtiEnvEventEnable オブジェクトの
       _event_callback_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」, ということ.
       つまり, コールバックを設定しているイベント一覧を示す JvmtiEventEnabled オブジェクトのこと.)
  
      enabling 引数が true であれば, 
      env->env_event_enable()->_event_callback_enabled の値を
      「現在の値に event_type 引数で指定された種別を示すビットを加えた値」に変更する.
  
      逆に enabling 引数が false であれば, 
      
      「現在の値から event_type 引数で指定された種別を示すビットをクリアした値」に変更する.
  
      ---------------------------------------- -}

	  // update the callback enable/disable bit
	  jlong enabled_bits = env->env_event_enable()->_event_callback_enabled.get_bits();
	  jlong bit_for = JvmtiEventEnabled::bit_for(event_type);
	  if (enabling) {
	    enabled_bits |= bit_for;
	  } else {
	    enabled_bits &= ~bit_for;
	  }
	  env->env_event_enable()->_event_callback_enabled.set_bits(enabled_bits);
	
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::recompute_enabled() を呼んで, 
      "truly enabled event" 情報を更新する.
      ---------------------------------------- -}

	  recompute_enabled();
	}
	
```


