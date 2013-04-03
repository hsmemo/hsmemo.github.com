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
void PSParallelCompact::compact() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("5 compact");
	  // trace("5");
	  TraceTime tm("compaction phase", print_phases(), true, gclog_or_tty);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	  PSOldGen* old_gen = heap->old_gen();
	  old_gen->start_array()->reset();
	  uint parallel_gc_threads = heap->gc_task_manager()->workers();
	  TaskQueueSetSuper* qset = ParCompactionManager::region_array();
	  ParallelTaskTerminator terminator(parallel_gc_threads, qset);
	
  {- -------------------------------------------
  (1) GCTaskQueue::create() で GCTaskQueue を作り, 
      そこに DrainStacksCompactionTask, UpdateDensePrefixTask, および 
      StealRegionCompactionTask をつめる.
  
      (See: PSParallelCompact::enqueue_region_draining_tasks(), 
            PSParallelCompact::enqueue_dense_prefix_tasks(), 
            PSParallelCompact::enqueue_region_stealing_tasks())
      ---------------------------------------- -}

	  GCTaskQueue* q = GCTaskQueue::create();
	  enqueue_region_draining_tasks(q, parallel_gc_threads);
	  enqueue_dense_prefix_tasks(q, parallel_gc_threads);
	  enqueue_region_stealing_tasks(q, &terminator, parallel_gc_threads);
	
	  {

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    TraceTime tm_pc("par compact", print_phases(), true, gclog_or_tty);
	
  {- -------------------------------------------
  (1) 最後に, 以上の処理が終了するまで待機するための WaitForBarrierGCTask も追加しておく.
      ---------------------------------------- -}

	    WaitForBarrierGCTask* fin = WaitForBarrierGCTask::create();
	    q->enqueue(fin);
	
  {- -------------------------------------------
  (1) GCTaskManager::add_list() で, GCTask をキューに追加して GCTaskThread 達に実行させる.
      ---------------------------------------- -}

	    gc_task_manager()->add_list(q);
	
  {- -------------------------------------------
  (1) WaitForBarrierGCTask::wait_for() で GCTask が全て実行されるまで待機する.
      待機が解けたら, WaitForBarrierGCTask::destroy() で WaitForBarrierGCTask オブジェクトを破棄する.
      ---------------------------------------- -}

	    fin->wait_for();
	
	    // We have to release the barrier tasks!
	    WaitForBarrierGCTask::destroy(fin);
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	#ifdef  ASSERT
	    // Verify that all regions have been processed before the deferred updates.
	    // Note that perm_space_id is skipped; this type of verification is not
	    // valid until the perm gen is compacted by regions.
	    for (unsigned int id = old_space_id; id < last_space_id; ++id) {
	      verify_complete(SpaceId(id));
	    }
	#endif
	  }
	
  {- -------------------------------------------
  (1) PSParallelCompact::update_deferred_objects() を呼んで, 
      複数 region にまたがっていたために
      ポインタの修正が行えていないオブジェクトに対して処理を行う.
      (なお, ここで使用する ParCompactionManager は何でもいいので, 適当に 0 番のものを使っている)
      ---------------------------------------- -}

	  {
	    // Update the deferred objects, if any.  Any compaction manager can be used.
	    TraceTime tm_du("deferred updates", print_phases(), true, gclog_or_tty);
	    ParCompactionManager* cm = ParCompactionManager::manager_array(0);
	    for (unsigned int id = old_space_id; id < last_space_id; ++id) {
	      update_deferred_objects(cm, SpaceId(id));
	    }
	  }
	}
	
```


