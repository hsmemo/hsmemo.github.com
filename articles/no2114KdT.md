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
Java_sun_management_MemoryPoolImpl_setCollectionThreshold0
  (JNIEnv *env, jobject pool, jlong current, jlong newThreshold)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_SetPoolThreshold() を呼んで, collection usage threshold の 
      high threshold 及び low threshold を変更する.
  
      (なお, high threshold と low threshold の両方を指定された値に設定する.
       high が low を下回る瞬間があるとまずいので, 現在の閾値と指定された閾値の大小を見て, 
       どちらを先に変更するかを変えている.)
      ---------------------------------------- -}

	    // Set both high and low threshold to the same threshold
	    if (newThreshold > current) {
	        // high threshold has to be set first so that high >= low
	        jmm_interface->SetPoolThreshold(env, pool,
	                                        JMM_COLLECTION_USAGE_THRESHOLD_HIGH,
	                                        newThreshold);
	        jmm_interface->SetPoolThreshold(env, pool,
	                                        JMM_COLLECTION_USAGE_THRESHOLD_LOW,
	                                        newThreshold);
	    } else {
	        // low threshold has to be set first so that high >= low
	        jmm_interface->SetPoolThreshold(env, pool,
	                                        JMM_COLLECTION_USAGE_THRESHOLD_LOW,
	                                        newThreshold);
	        jmm_interface->SetPoolThreshold(env, pool,
	                                        JMM_COLLECTION_USAGE_THRESHOLD_HIGH,
	                                        newThreshold);
	    }
	}
	
```


