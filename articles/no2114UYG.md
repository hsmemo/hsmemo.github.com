---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/HotspotThread.c

### 名前(function name)
```
JNIEXPORT jint JNICALL
Java_sun_management_HotspotThread_getInternalThreadTimes0
  (JNIEnv *env, jobject dummy, jobjectArray names, jobjectArray times)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_GetInternalThreadTimes() を呼び出すだけ.
      ---------------------------------------- -}

	    return jmm_interface->GetInternalThreadTimes(env, names, times);
	}
	
```


