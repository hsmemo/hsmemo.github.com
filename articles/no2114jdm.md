---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/HotSpotDiagnostic.c

### 名前(function name)
```
JNIEXPORT void JNICALL
Java_sun_management_HotSpotDiagnostic_dumpHeap
  (JNIEnv *env, jobject dummy, jstring outputfile, jboolean live)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_DumpHeap0() を呼び出すだけ.
      ---------------------------------------- -}

	    jmm_interface->DumpHeap0(env, outputfile, live);
	}
	
```


