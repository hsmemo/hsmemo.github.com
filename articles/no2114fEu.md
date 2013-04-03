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
Java_sun_management_ThreadImpl_getThreadUserCpuTime1
  (JNIEnv *env, jclass cls, jlongArray ids, jlongArray timeArray)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_GetThreadCpuTimesWithKind() を呼び出すだけ.
      ---------------------------------------- -}

	    jmm_interface->GetThreadCpuTimesWithKind(env, ids, timeArray,
	                                             JNI_FALSE /* user */);
	}
	
```


