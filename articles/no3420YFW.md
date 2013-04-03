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
ThreadCritical::ThreadCritical() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  pthread_t self = pthread_self();

  {- -------------------------------------------
  (1) 現在ロックを保持しているのがカレントスレッドでなければ, 
      pthread_mutex_lock() でロックの確保を行う
      (必要ならばここでブロック).
      ---------------------------------------- -}

	  if (self != tc_owner) {
	    int ret = pthread_mutex_lock(&tc_mutex);
	    guarantee(ret == 0, "fatal error with pthread_mutex_lock()");
	    assert(tc_count == 0, "Lock acquired with illegal reentry count.");
	    tc_owner = self;
	  }

  {- -------------------------------------------
  (1) ロックの再帰確保数をインクリメントしておく.
      ---------------------------------------- -}

	  tc_count++;
	}
	
```


