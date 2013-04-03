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
ConcurrentMarkThread::ConcurrentMarkThread(ConcurrentMark* cm) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化
      ---------------------------------------- -}

	  ConcurrentGCThread(),

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _cm(cm),
	  _started(false),
	  _in_progress(false),
	  _vtime_accum(0.0),
	  _vtime_mark_accum(0.0),
	  _vtime_count_accum(0.0)
	{

  {- -------------------------------------------
  (1) ConcurrentGCThread::create_and_start() を呼んで, 
      新しいスレッドを作り, 実行を開始させる
      ---------------------------------------- -}

	  create_and_start();
	}
	
```


