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
// initiating_loader - NULL is a valid value, must be checked
// class_count_ptr - pre-checked for NULL
// classes_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetClassLoaderClasses(jobject initiating_loader, jint* class_count_ptr, jclass** classes_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiGetLoadedClasses::getClassLoaderClasses() を呼び出すだけ.
      ---------------------------------------- -}

	  return JvmtiGetLoadedClasses::getClassLoaderClasses(this, initiating_loader,
	                                                  class_count_ptr, classes_ptr);
	} /* end GetClassLoaderClasses */
	
```


