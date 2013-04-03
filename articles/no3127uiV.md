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
void CMTask::drain_global_stack(bool partially) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 何らかの理由でこの CMTask が中断されていたら, ここでリターン
      ---------------------------------------- -}

	  if (has_aborted())
	    return;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // We have a policy to drain the local queue before we attempt to
	  // drain the global stack.
	  assert(partially || _task_queue->size() == 0, "invariant");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (target_size は, 今回の処理で _task_queue 内の要素をどこまで減らすかを表す.
      partially 引数が true なら partial_mark_stack_size_target() で計算する.
      partially 引数が false なら, すべて処理し終える.)
      ---------------------------------------- -}

	  // Decide what the target size is, depending whether we're going to
	  // drain it partially (so that other tasks can steal if they run out
	  // of things to do) or totally (at the very end).  Notice that,
	  // because we move entries from the global stack in chunks or
	  // because another task might be doing the same, we might in fact
	  // drop below the target. But, this is not a problem.
	  size_t target_size;
	  if (partially)
	    target_size = _cm->partial_mark_stack_size_target();
	  else
	    target_size = 0;
	
  {- -------------------------------------------
  (1) (以降の処理で,
      _cm->mark_stack_size() 内の要素数が目標数(target_size)以下になるまで処理を続ける)
      ---------------------------------------- -}

	  if (_cm->mark_stack_size() > target_size) {

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] draining global_stack, target size %d",
	                             _task_id, target_size);
	
    {- -------------------------------------------
  (1.1) ... するまで,
        CMTask::get_entries_from_global_stack() で取ってきて
        CMTask::drain_local_queue() で処理する,
        という作業を続ける.
        ---------------------------------------- -}

	    while (!has_aborted() && _cm->mark_stack_size() > target_size) {
	      get_entries_from_global_stack();
	      drain_local_queue(partially);
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (_cm->verbose_low())
	      gclog_or_tty->print_cr("[%d] drained global stack, size = %d",
	                             _task_id, _cm->mark_stack_size());
	  }
	}
	
```


