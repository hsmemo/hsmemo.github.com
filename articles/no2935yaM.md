---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp
### 説明(description)

```
// For the specified env: compute the currently truly enabled events
// set external state accordingly.
// Return value and set value must include all events.
// But outside this class, only non-thread-filtered events can be queried..
```

### 名前(function name)
```
jlong
JvmtiEventControllerPrivate::recompute_env_enabled(JvmtiEnvBase* env) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力用の処理)
      ---------------------------------------- -}

	  jlong was_enabled = env->env_event_enable()->_event_enabled.get_bits();

  {- -------------------------------------------
  (1) 「env 引数が表す Jvmti environment 中で, 
       ユーザーが有効と指定したイベントでかつ callback も設定されているもの」を表す bit mask を作る.
      ---------------------------------------- -}

	  jlong now_enabled =
	    env->env_event_enable()->_event_callback_enabled.get_bits() &
	    env->env_event_enable()->_event_user_enabled.get_bits();
	
  {- -------------------------------------------
  (1) 上記の bit mask から, 現在の phase では対応していないものをクリア(filtering)する
      ---------------------------------------- -}

	  switch (JvmtiEnv::get_phase()) {
	  case JVMTI_PHASE_PRIMORDIAL:
	  case JVMTI_PHASE_ONLOAD:
	    // only these events allowed in primordial or onload phase
	    now_enabled &= (EARLY_EVENT_BITS & ~THREAD_FILTERED_EVENT_BITS);
	    break;
	  case JVMTI_PHASE_START:
	    // only these events allowed in start phase
	    now_enabled &= EARLY_EVENT_BITS;
	    break;
	  case JVMTI_PHASE_LIVE:
	    // all events allowed during live phase
	    break;
	  case JVMTI_PHASE_DEAD:
	    // no events allowed when dead
	    now_enabled = 0;
	    break;
	  default:
	    assert(false, "no other phases - sanity check");
	    break;
	  }
	
  {- -------------------------------------------
  (1) 作成した bit mask を env->env_event_enable()->_event_enabled に記録する.
  
      (これは, 「env 引数で指定された JvmtiEnvBase オブジェクト中の JvmtiEnvEventEnable オブジェクトの
       _event_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」, ということ.)
      ---------------------------------------- -}

	  // will we really send these events to this env
	  env->env_event_enable()->_event_enabled.set_bits(now_enabled);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  trace_changed(now_enabled, (now_enabled ^ was_enabled)  & ~THREAD_FILTERED_EVENT_BITS);
	
  {- -------------------------------------------
  (1) 新しい truly enabled event をリターンする
      ---------------------------------------- -}

	  return now_enabled;
	}
	
```


