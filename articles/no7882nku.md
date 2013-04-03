---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepThread.cpp
### 説明(description)
(なおコメントによると, 
このメソッドは ConcurrentMarkSweepThread のメソッドだが JavaThread からしか呼び出されない.
現状では, 起動時に main/Primordial (Java)Thread から呼び出されるだけ.
将来的には CMS thread 自身が SurrogateLockerThread を作るようにすべきか検討する, 
とのこと.)

```
// Note: this method, although exported by the ConcurrentMarkSweepThread,
// which is a non-JavaThread, can only be called by a JavaThread.
// Currently this is done at vm creation time (post-vm-init) by the
// main/Primordial (Java)Thread.
// XXX Consider changing this in the future to allow the CMS thread
// itself to create this thread?
```

### 名前(function name)
```
void ConcurrentMarkSweepThread::makeSurrogateLockerThread(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(UseConcMarkSweepGC, "SLT thread needed only for CMS GC");
	  assert(_slt == NULL, "SLT already created");

  {- -------------------------------------------
  (1) SurrogateLockerThread::make() で SurrogateLockerThread を生成し, _slt フィールドに格納する.
      ---------------------------------------- -}

	  _slt = SurrogateLockerThread::make(THREAD);
	}
	
```


