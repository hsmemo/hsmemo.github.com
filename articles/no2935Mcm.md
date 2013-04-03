---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
void ConcurrentMark::markFromRoots() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下のような assert を入れたいが成り立たないので今は入れてない, とのこと)
      ---------------------------------------- -}

	  // we might be tempted to assert that:
	  // assert(asynch == !SafepointSynchronize::is_at_safepoint(),
	  //        "inconsistent argument?");
	  // However that wouldn't be right, because it's possible that
	  // a safepoint is indeed in progress as a younger generation
	  // stop-the-world GC happens even as we mark in this generation.
	
  {- -------------------------------------------
  (1) _restart_for_overflow フィールドを初期化しておく.
      (以降の処理で失敗すると true に変わる)
      ---------------------------------------- -}

	  _restart_for_overflow = false;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  size_t active_workers = MAX2((size_t) 1, parallel_marking_threads());

  {- -------------------------------------------
  (1) (デバッグ用の処理) (See: ForceOverflowSettings)
      ---------------------------------------- -}

	  force_overflow_conc()->init();

  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  set_phase(active_workers, true /* concurrent */);
	
  {- -------------------------------------------
  (1) CMConcurrentMarkingTask::work() を呼び出して, 
      root から辿れる範囲の Concurrent marking 処理を行う.
    
      (なお, ConcurrentMark::parallel_marking_threads() が 1 以上の場合は, 
       WorkGang::run_task() 経由で呼び出す.
       これにより, 処理がマルチスレッドで実行される)
      ---------------------------------------- -}

	  CMConcurrentMarkingTask markingTask(this, cmThread());
	  if (parallel_marking_threads() > 0)
	    _parallel_workers->run_task(&markingTask);
	  else
	    markingTask.work(0);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  print_stats();
	}
	
```


