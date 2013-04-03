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
// capabilities_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::RelinquishCapabilities(const jvmtiCapabilities* capabilities_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiManageCapabilities::relinquish_capabilities() を呼び出すだけ.
  
      (なお, get_capabilities() は _current_capabilities フィールドを指すポインタ.
       (See: JvmtiEnvBase::get_capabilities()))
      ---------------------------------------- -}

	  JvmtiManageCapabilities::relinquish_capabilities(get_capabilities(), capabilities_ptr, get_capabilities());
	  return JVMTI_ERROR_NONE;
	} /* end RelinquishCapabilities */
	
```


