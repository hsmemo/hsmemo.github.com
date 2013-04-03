---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
void
G1CollectedHeap::doConcurrentMark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ConcurrentMarkThread::set_started() を呼んで
      ConcurrentMarkThread::_started フィールドを true にした後)
      CGC_lock に対して Monitor::notify() を呼んで ConcurrentMarkThread スレッドを起床させる.
      (See: ConcurrentMarkThread::sleepBeforeNextCycle())
    
      (ただし, 今現在 ConcurrentMarkThread が処理を行っている途中であれば
       (= ConcurrentMarkThread::in_progress() が true であれば), 何もしない)
  
      (なお, これらの操作は CGC_lock で排他した状態で行う)
      ---------------------------------------- -}

	  MutexLockerEx x(CGC_lock, Mutex::_no_safepoint_check_flag);
	  if (!_cmThread->in_progress()) {
	    _cmThread->set_started();
	    CGC_lock->notify();
	  }
	}
	
```


