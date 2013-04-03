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
// register extension functions and events. In this implementation we
// have a single extension function (to prove the API) that tests if class
// unloading is enabled or disabled. We also have a single extension event
// EXT_EVENT_CLASS_UNLOAD which is used to provide the JVMDI_EVENT_CLASS_UNLOAD
// event. The function and the event are registered here.
//
```

### 名前(function name)
```
void JvmtiExtensions::register_extensions() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (フィールドの初期化)
      ---------------------------------------- -}

	  _ext_functions = new (ResourceObj::C_HEAP) GrowableArray<jvmtiExtensionFunctionInfo*>(1,true);
	  _ext_events = new (ResourceObj::C_HEAP) GrowableArray<jvmtiExtensionEventInfo*>(1,true);
	
  {- -------------------------------------------
  (1) (以下が extension function の登録処理)
      ---------------------------------------- -}

	  // register our extension function

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	  static jvmtiParamInfo func_params[] = {
	    { (char*)"IsClassUnloadingEnabled", JVMTI_KIND_OUT,  JVMTI_TYPE_JBOOLEAN, JNI_FALSE }
	  };
	  static jvmtiExtensionFunctionInfo ext_func = {
	    (jvmtiExtensionFunction)IsClassUnloadingEnabled,
	    (char*)"com.sun.hotspot.functions.IsClassUnloadingEnabled",
	    (char*)"Tell if class unloading is enabled (-noclassgc)",
	    sizeof(func_params)/sizeof(func_params[0]),
	    func_params,
	    0,              // no non-universal errors
	    NULL
	  };

    {- -------------------------------------------
  (1.1) (フィールドの初期化)
        (_ext_functions に値をセット)
        ---------------------------------------- -}

	  _ext_functions->append(&ext_func);
	
  {- -------------------------------------------
  (1) (以下が extension event の登録処理)
      ---------------------------------------- -}

	  // register our extension event
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	  static jvmtiParamInfo event_params[] = {
	    { (char*)"JNI Environment", JVMTI_KIND_IN, JVMTI_TYPE_JNIENV, JNI_FALSE },
	    { (char*)"Thread", JVMTI_KIND_IN, JVMTI_TYPE_JTHREAD, JNI_FALSE },
	    { (char*)"Class", JVMTI_KIND_IN, JVMTI_TYPE_JCLASS, JNI_FALSE }
	  };
	  static jvmtiExtensionEventInfo ext_event = {
	    EXT_EVENT_CLASS_UNLOAD,
	    (char*)"com.sun.hotspot.events.ClassUnload",
	    (char*)"CLASS_UNLOAD event",
	    sizeof(event_params)/sizeof(event_params[0]),
	    event_params
	  };

    {- -------------------------------------------
  (1.1) (フィールドの初期化)
        (_ext_events に値をセット)
        ---------------------------------------- -}

	  _ext_events->append(&ext_event);
	}
	
```


