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
void DrainStacksCompactionTask::do_it(GCTaskManager* manager, uint which) {
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

	  NOT_PRODUCT(TraceTime tm("DrainStacksCompactionTask",
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

	  // Process any regions already in the compaction managers stacks.
	  cm->drain_region_stacks();
	}
	
```


