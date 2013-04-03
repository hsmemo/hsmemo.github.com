---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _jni_environment フィールドのアドレスを返すだけ.
      ---------------------------------------- -}

	  // Returns the jni environment for this thread
	  JNIEnv* jni_environment()                      { return &_jni_environment; }
	
```


