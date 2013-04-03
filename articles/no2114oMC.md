---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/pcTasks.cpp

### 名前(function name)
```
void StealRegionCompactionTask::do_it(GCTaskManager* manager, uint which) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Universe::heap()->is_gc_active(), "called outside gc");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  NOT_PRODUCT(TraceTime tm("StealRegionCompactionTask",
	    PrintGCDetails && TraceParallelOldGCTasks, true, gclog_or_tty));
	
  {- -------------------------------------------
  (1) ParCompactionManager::gc_thread_compaction_manager() で, 
      これを実行している GCTaskThread 用の ParCompactionManager オブジェクトを取得する.
      ---------------------------------------- -}

	  ParCompactionManager* cm =
	    ParCompactionManager::gc_thread_compaction_manager(which);
	
  {- -------------------------------------------
  (1) ParCompactionManager::drain_region_stacks() を呼んで, 
      ParCompactionManager 内のスタックに残っている全ての region に対して
      オブジェクトの移動処理を行う.
      ---------------------------------------- -}

	  // Has to drain stacks first because there may be regions on
	  // preloaded onto the stack and this thread may never have
	  // done a draining task.  Are the draining tasks needed?
	
	  cm->drain_region_stacks();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  size_t region_index = 0;
	  int random_seed = 17;
	
	  // If we're the termination task, try 10 rounds of stealing before
	  // setting the termination flag
	
	  while(true) {
	    if (ParCompactionManager::steal(which, &random_seed, region_index)) {
	      PSParallelCompact::fill_and_update_region(cm, region_index);
	      cm->drain_region_stacks();
	    } else {
	      if (terminator()->offer_termination()) {
	        break;
	      }
	      // Go around again.
	    }
	  }
	  return;
	}
	
```


