---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/MemoryPoolImpl.c

### 名前(function name)
```
JNIEXPORT void JNICALL
Java_sun_management_MemoryPoolImpl_setPoolCollectionSensor
  (JNIEnv *env, jobject pool, jobject sensor)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_SetPoolSensor() を呼び出すだけ.
      ---------------------------------------- -}

	    jmm_interface->SetPoolSensor(env, pool,
	                                 JMM_COLLECTION_USAGE_THRESHOLD_HIGH, sensor);
	}
	
```


