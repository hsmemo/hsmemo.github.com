---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/ThreadImpl.c

### 名前(function name)
```
JNIEXPORT void JNICALL
Java_sun_management_ThreadImpl_getThreadAllocatedMemory1
  (JNIEnv *env, jclass cls, jlongArray ids, jlongArray sizeArray)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_GetThreadAllocatedMemory() を呼び出すだけ.
      ---------------------------------------- -}

	    jmm_interface->GetThreadAllocatedMemory(env, ids, sizeArray);
	}
	
```


