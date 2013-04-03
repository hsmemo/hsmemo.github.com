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
  JvmtiGetLoadedClassesClosure() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    JvmtiGetLoadedClassesClosure* that = get_this();
	    assert(that == NULL, "JvmtiGetLoadedClassesClosure in use");

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	    _initiatingLoader = NULL;
	    _count = 0;
	    _list = NULL;
	    _index = 0;

  {- -------------------------------------------
  (1) set_this() 関数を呼んで, 
      カレントスレッドの jvmti_get_loaded_classes_closure フィールド内に
      この JvmtiGetLoadedClassesClosure オブジェクトを登録しておく.
      ---------------------------------------- -}

	    set_this(this);
	  }
	
```


