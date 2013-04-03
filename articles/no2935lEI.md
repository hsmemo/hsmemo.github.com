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
void ConcurrentGCThread::stsLeave() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert( Thread::current()->is_ConcurrentGC_thread(),
	          "only a conc GC thread can call this" );

  {- -------------------------------------------
  (1) SuspendibleThreadSet::leave() を呼び出し, 
      ConcurrentGCThread::_sts に格納されている SuspendibleThreadSet オブジェクト内から
      カレントスレッドの登録を削除する
      (See: SuspendibleThreadSet)
      ---------------------------------------- -}

	  _sts.leave();
	}
	
```


