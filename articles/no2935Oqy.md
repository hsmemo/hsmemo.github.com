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
// extension_count_ptr - pre-checked for NULL
// extensions - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetExtensionEvents(jint* extension_count_ptr, jvmtiExtensionEventInfo** extensions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiExtensions::get_events() を呼び出すだけ.
      ---------------------------------------- -}

	  return JvmtiExtensions::get_events(this, extension_count_ptr, extensions);
	} /* end GetExtensionEvents */
	
```


