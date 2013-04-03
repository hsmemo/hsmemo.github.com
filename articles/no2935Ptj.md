---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.cpp

### 名前(function name)
```
ConcurrentG1RefineThread::
ConcurrentG1RefineThread(ConcurrentG1Refine* cg1r, ConcurrentG1RefineThread *next,
                         int worker_id_offset, int worker_id) :
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

	  _worker_id_offset(worker_id_offset),
	  _worker_id(worker_id),
	  _active(false),
	  _next(next),
	  _monitor(NULL),
	  _cg1r(cg1r),
	  _vtime_accum(0.0)
	{
	
  {- -------------------------------------------
  (1) (コメントによると, 
       各 ConcurrentG1RefineThread はそれぞれ個別の Monitor オブジェクトを保持する.
       ただし, 0 版目の ConcurrentG1RefineThread だけは DirtyCardQ_CBL_mon を使用する.
       各 ConcurrentG1RefineThread は, 一つ番号の小さい ConcurrentG1RefineThread によって起床される.
       起床のタイミングは, 処理が多くスレッド数が足りないと判断された場合.
       とのこと)
      ---------------------------------------- -}

	  // Each thread has its own monitor. The i-th thread is responsible for signalling
	  // to thread i+1 if the number of buffers in the queue exceeds a threashold for this
	  // thread. Monitors are also used to wake up the threads during termination.
	  // The 0th worker in notified by mutator threads and has a special monitor.
	  // The last worker is used for young gen rset size sampling.

  {- -------------------------------------------
  (1) _monitor フィールドを初期化
      (0 版目の ConcurrentG1RefineThread は DirtyCardQ_CBL_mon を使用.
       その他は新しい Monitor をこの場で作成して使用)
      ---------------------------------------- -}

	  if (worker_id > 0) {
	    _monitor = new Monitor(Mutex::nonleaf, "Refinement monitor", true);
	  } else {
	    _monitor = DirtyCardQ_CBL_mon;
	  }

  {- -------------------------------------------
  (1) ConcurrentG1RefineThread::initialize() を呼んで, 
      フィールドの初期化を行う.
      ---------------------------------------- -}

	  initialize();

  {- -------------------------------------------
  (1) ConcurrentGCThread::create_and_start() を呼んで, 
      新しいスレッドを作り, 実行を開始させる
      ---------------------------------------- -}

	  create_and_start();
	}
	
```


