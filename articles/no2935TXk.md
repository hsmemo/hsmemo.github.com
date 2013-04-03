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
  void extract(JvmtiEnv *env, jclass* result) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 取得した結果を result 引数で指定された配列に格納する.
      (要素を JvmtiGetLoadedClassesClosure::get_element() で取得し, 
       JNI Handle 化して格納している)
      ---------------------------------------- -}

	    for (int index = 0; index < _count; index += 1) {
	      result[index] = (jclass) env->jni_reference(get_element(index));
	    }
	  }
	
```


