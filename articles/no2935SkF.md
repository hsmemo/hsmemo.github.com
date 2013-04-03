---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.cpp

### 名前(function name)
```
  static void set_this(JvmtiGetLoadedClassesClosure* that) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread::set_jvmti_get_loaded_classes_closure() を呼んで, 
      カレントスレッドの jvmti_get_loaded_classes_closure フィールドに 
      that 引数のオブジェクトをセットする.
      ---------------------------------------- -}

	    JavaThread* thread = JavaThread::current();
	    thread->set_jvmti_get_loaded_classes_closure(that);
	  }
	
```


