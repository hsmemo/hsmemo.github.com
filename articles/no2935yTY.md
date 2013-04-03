---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// callbacks - NULL is a valid value, must be checked
// size_of_callbacks - pre-checked to be greater than or equal to 0
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::SetEventCallbacks(const jvmtiEventCallbacks* callbacks, jint size_of_callbacks) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventController::set_event_callbacks() を呼び出すだけ.
      ---------------------------------------- -}

	  JvmtiEventController::set_event_callbacks(this, callbacks, size_of_callbacks);
	  return JVMTI_ERROR_NONE;
	} /* end SetEventCallbacks */
	
```


