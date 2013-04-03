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
Java_sun_management_ThreadImpl_setThreadContentionMonitoringEnabled0
  (JNIEnv *env, jclass cls, jboolean flag)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_SetBoolAttribute() を呼び出すだけ.
      ---------------------------------------- -}

	    jmm_interface->SetBoolAttribute(env, JMM_THREAD_CONTENTION_MONITORING, flag);
	}
	
```


