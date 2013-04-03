---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// Threads_lock NOT held, java_thread not protected by lock
// java_thread - pre-checked
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::ForceEarlyReturnInt(JavaThread* java_thread, jint value) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEnvBase::force_early_return() を呼び出すだけ.
      ---------------------------------------- -}

	  jvalue val;
	  val.i = value;
	  return force_early_return(java_thread, val, itos);
	} /* end ForceEarlyReturnInt */
	
```


