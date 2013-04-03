---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/MemoryImpl.c

### 名前(function name)
```
JNIEXPORT jobject JNICALL Java_sun_management_MemoryImpl_getMemoryManagers0
  (JNIEnv *env, jclass dummy) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    return jmm_interface->GetMemoryManagers(env, NULL);
	}
	
```


