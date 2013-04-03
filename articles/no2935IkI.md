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
JvmtiAgentThread::JvmtiAgentThread(JvmtiEnv* env, jvmtiStartFunction start_fn, const void *start_arg)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread::JavaThread() を呼び出す
      ---------------------------------------- -}

	    : JavaThread(start_function_wrapper) {

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	    _env = env;
	    _start_fn = start_fn;
	    _start_arg = start_arg;
	}
	
```


