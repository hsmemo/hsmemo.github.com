---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp

### 名前(function name)
```
void ConcurrentGCThread::wait_for_universe_init() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) is_init_completed() が true になるまでここで待機する.
      (待機処理は CGC_lock に対して Monitor::wait() することで行う.
       起床処理は G1CollectedHeap::doConcurrentMark())
      ---------------------------------------- -}

	  MutexLockerEx x(CGC_lock, Mutex::_no_safepoint_check_flag);
	  while (!is_init_completed() && !_should_terminate) {
	    CGC_lock->wait(Mutex::_no_safepoint_check_flag, 200);
	  }
	}
	
```


