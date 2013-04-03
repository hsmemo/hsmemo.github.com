---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/threadCritical_linux.cpp

### 名前(function name)
```
ThreadCritical::~ThreadCritical() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(tc_owner == pthread_self(), "must have correct owner");
	  assert(tc_count > 0, "must have correct count");
	
  {- -------------------------------------------
  (1) ロックの再帰確保数をデクリメントする.
      もし確保数が 0 になったら, pthread_mutex_unlock() でロックを解放する.
      ---------------------------------------- -}

	  tc_count--;
	  if (tc_count == 0) {
	    tc_owner = 0;
	    int ret = pthread_mutex_unlock(&tc_mutex);
	    guarantee(ret == 0, "fatal error with pthread_mutex_unlock()");
	  }
	}
	
```


