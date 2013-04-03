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
// For the specified env and thread: compute the currently truly enabled events
// set external state accordingly.  Only thread-filtered events are included.
```

### 名前(function name)
```
jlong
JvmtiEventControllerPrivate::recompute_env_thread_enabled(JvmtiEnvThreadState* ets, JvmtiThreadState* state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JvmtiEnv *env = ets->get_env();
	
  {- -------------------------------------------
  (1) 現在の　truly enabled event　設定を取得
      ---------------------------------------- -}

	  jlong was_enabled = ets->event_enable()->_event_enabled.get_bits();

  {- -------------------------------------------
  (1) (以降で, 新しい truly enabled event を計算する)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) まず, 
       「ets 引数が表す JvmtiEnvThreadState 内または ets に対応する JvmtiEnv (env 変数) 内で
       ユーザーが有効化しているイベント通知種別であり, 
       かつ callback も設定されており, かつ thread filtered event であるもの」
       を表す bit mask を作る
      ---------------------------------------- -}

	  jlong now_enabled =  THREAD_FILTERED_EVENT_BITS &
	    env->env_event_enable()->_event_callback_enabled.get_bits() &
	    (env->env_event_enable()->_event_user_enabled.get_bits() |
	     ets->event_enable()->_event_user_enabled.get_bits());
	
    {- -------------------------------------------
  (1.1) ただし, frame pops と field watchs に関しては, 監視対象が一つも設定されてなければ bit mask からは外す.
        ---------------------------------------- -}

	  // for frame pops and field watchs, computed enabled state
	  // is only true if an event has been requested
	  if (!ets->has_frame_pops()) {
	    now_enabled &= ~FRAME_POP_BIT;
	  }
	  if (*((int *)JvmtiExport::get_field_access_count_addr()) == 0) {
	    now_enabled &= ~FIELD_ACCESS_BIT;
	  }
	  if (*((int *)JvmtiExport::get_field_modification_count_addr()) == 0) {
	    now_enabled &= ~FIELD_MODIFICATION_BIT;
	  }
	
    {- -------------------------------------------
  (1.1) また, phase が JVMTI_PHASE_DEAD であれば 
        (この状態ではもうイベント通知は許可されないので) 
        bit mask から要素を全部外す(空にする).
        ---------------------------------------- -}

	  switch (JvmtiEnv::get_phase()) {
	  case JVMTI_PHASE_DEAD:
	    // no events allowed when dead
	    now_enabled = 0;
	    break;
	  }
	
  {- -------------------------------------------
  (1) もし新しい truly enabled event が古いものと異なれば, 以下の if ブロック内でその変更を反映させる処理を行う.
      ---------------------------------------- -}

	  // if anything changed do update
	  if (now_enabled != was_enabled) {
	
    {- -------------------------------------------
  (1.1) 新しい bit mask を ets->event_enable()->_event_enabled に記録しておく.
  
         (これは, 「ets 引数で指定された JvmtiEnvThreadState オブジェクト中の JvmtiEnvThreadEventEnable オブジェクトの
          _event_enabled フィールドに格納されている JvmtiEventEnabled オブジェクト」, ということ.)
        ---------------------------------------- -}

	    // will we really send these events to this thread x env
	    ets->event_enable()->_event_enabled.set_bits(now_enabled);
	
    {- -------------------------------------------
  (1.1) もし single step か breakpoint が変わっていたのであれば,
        JvmtiEnvThreadState::reset_current_location() を呼んで ...#TODO
        ---------------------------------------- -}

	    // If the enabled status of the single step or breakpoint events changed,
	    // the location status may need to change as well.
	    jlong changed = now_enabled ^ was_enabled;
	    if (changed & SINGLE_STEP_BIT) {
	      ets->reset_current_location(JVMTI_EVENT_SINGLE_STEP, (now_enabled & SINGLE_STEP_BIT) != 0);
	    }
	    if (changed & BREAKPOINT_BIT) {
	      ets->reset_current_location(JVMTI_EVENT_BREAKPOINT,  (now_enabled & BREAKPOINT_BIT) != 0);
	    }

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    trace_changed(state, now_enabled, changed);
	  }

  {- -------------------------------------------
  (1) 新しい truly enabled event をリターンする
      ---------------------------------------- -}

	  return now_enabled;
	}
	
```


