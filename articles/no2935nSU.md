---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExtensions.cpp
### 説明(description)

```
// set callback for an extension event and enable/disable it.

```

### 名前(function name)
```
jvmtiError JvmtiExtensions::set_event_callback(JvmtiEnv* env,
                                               jint extension_event_index,
                                               jvmtiExtensionEvent callback)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(_ext_events != NULL, "registration not done");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiExtensionEventInfo* event = NULL;
	
  {- -------------------------------------------
  (1) extension_event_index 引数で指定された index が
      実際に _ext_events 中に存在していることを確認する
      (そして対応するイベントを event 変数にセットしておく).
      
      もし存在しない index だった場合は, ここでリターン(JVMTI_ERROR_ILLEGAL_ARGUMENT).
      ---------------------------------------- -}

	  // if there are extension events registered then validate that the
	  // extension_event_index matches one of the registered events.
	  if (_ext_events != NULL) {
	    for (int i=0; i<_ext_events->length(); i++ ) {
	      if (_ext_events->at(i)->extension_event_index == extension_event_index) {
	         event = _ext_events->at(i);
	         break;
	      }
	    }
	  }
	
	  // invalid event index
	  if (event == NULL) {
	    return JVMTI_ERROR_ILLEGAL_ARGUMENT;
	  }
	
  {- -------------------------------------------
  (1) JvmtiEventController::set_extension_event_callback() を呼んで, コールバックを設定する.
      ---------------------------------------- -}

	  JvmtiEventController::set_extension_event_callback(env, extension_event_index,
	                                                     callback);
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	}
	
```


