---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
void
JvmtiAgentThread::call_start_function() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コンストラクタで指定された関数(_start_fn)を
      コンストラクタで指定された値(_start_arg)を引数として呼び出すだけ.
      ---------------------------------------- -}

	    ThreadToNativeFromVM transition(this);
	    _start_fn(_env->jvmti_external(), jni_environment(), (void*)_start_arg);
	}
	
```


