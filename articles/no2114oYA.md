---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/VMManagementImpl.c

### 名前(function name)
```
JNIEXPORT jlong JNICALL
Java_sun_management_VMManagementImpl_getUnloadedClassSize
  (JNIEnv *env, jobject dummy)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JMM_CLASS_UNLOADED_BYTES を引数として, jmm_GetLongAttribute() を呼び出すだけ.
      ---------------------------------------- -}

	    return jmm_interface->GetLongAttribute(env, NULL,
	                                           JMM_CLASS_UNLOADED_BYTES);
	}
	
```


