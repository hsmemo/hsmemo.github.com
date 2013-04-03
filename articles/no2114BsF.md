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
JNIEXPORT jboolean JNICALL
Java_sun_management_VMManagementImpl_getVerboseClass
  (JNIEnv *env, jobject dummy)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JMM_VERBOSE_CLASS を引数として, jmm_GetBoolAttribute() を呼び出すだけ.
      ---------------------------------------- -}

	    return jmm_interface->GetBoolAttribute(env, JMM_VERBOSE_CLASS);
	}
	
```


