---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp

### 名前(function name)
```
void PSParallelCompact::enqueue_region_stealing_tasks(
                                     GCTaskQueue* q,
                                     ParallelTaskTerminator* terminator_ptr,
                                     uint parallel_gc_threads) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TraceTime tm("steal task setup", print_phases(), true, gclog_or_tty);
	
  {- -------------------------------------------
  (1) 引数で指定された個数分(以下 parallel_gc_threads 個分)だけ StealRegionCompactionTask を追加.
      ---------------------------------------- -}

	  // Once a thread has drained it's stack, it should try to steal regions from
	  // other threads.
	  if (parallel_gc_threads > 1) {
	    for (uint j = 0; j < parallel_gc_threads; j++) {
	      q->enqueue(new StealRegionCompactionTask(terminator_ptr));
	    }
	  }
	}
	
```


