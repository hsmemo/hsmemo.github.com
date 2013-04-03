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
SurrogateLockerThread::SurrogateLockerThread() :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread のコンストラクタを呼び出す (エントリポイントは _sltLoop())
      ---------------------------------------- -}

	  JavaThread(&_sltLoop),

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _monitor(Mutex::nonleaf, "SLTMonitor"),
	  _buffer(empty)
	{}
	
```


