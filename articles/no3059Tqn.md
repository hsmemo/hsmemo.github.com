---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/osThread_linux.cpp

### 名前(function name)
```
void OSThread::pd_initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(this != NULL, "check");

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _thread_id        = 0;
	  _pthread_id       = 0;
	  _siginfo = NULL;
	  _ucontext = NULL;
	  _expanding_stack = 0;
	  _alt_sig_stack = NULL;
	
	  sigemptyset(&_caller_sigmask);
	
	  _startThread_lock = new Monitor(Mutex::event, "startThread_lock", true);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_startThread_lock !=NULL, "check");
	}
	
```


