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
JNIEXPORT jobjectArray JNICALL
Java_sun_management_ThreadImpl_dumpThreads0
  (JNIEnv *env, jclass cls, jlongArray ids, jboolean lockedMonitors, jboolean lockedSynchronizers)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_DumpThreads() を呼び出すだけ.
      ---------------------------------------- -}

	    return jmm_interface->DumpThreads(env, ids, lockedMonitors, lockedSynchronizers);
	}
	
```


