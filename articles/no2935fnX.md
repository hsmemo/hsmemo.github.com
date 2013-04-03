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
// class_count_ptr - pre-checked for NULL
// classes_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetLoadedClasses(jint* class_count_ptr, jclass** classes_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiGetLoadedClasses::getLoadedClasses() を呼び出すだけ.
      ---------------------------------------- -}

	  return JvmtiGetLoadedClasses::getLoadedClasses(this, class_count_ptr, classes_ptr);
	} /* end GetLoadedClasses */
	
```


