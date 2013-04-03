---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.cpp

### 名前(function name)
```
void ConcurrentMarkThread::yield() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SuspendibleThreadSet::yield() を呼び出すだけ.
      ---------------------------------------- -}

	  _sts.yield("Concurrent Mark");
	}
	
```


