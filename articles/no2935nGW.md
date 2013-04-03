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
void ConcurrentMark::checkpointRootsFinalWork() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;
	  HandleMark   hm;
	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	
  {- -------------------------------------------
  (1) GC 処理の前に, CollectedHeap::ensure_parsability() を呼んで
      各 TLAB に残っている未使用領域を埋め, 
      オブジェクトでひとつながりの状態(= GC で辿れる状態)にしておく.
      ---------------------------------------- -}

	  g1h->ensure_parsability(false);
	
  {- -------------------------------------------
  (1) CMRemarkTask::work() で Final marking pause 処理を行う.
      (なお, G1CollectedHeap::use_parallel_gc_threads() が true の場合は, 
       WorkGang::run_task() 経由で呼び出す.
       これにより, 処理がマルチスレッドで実行される)
  
      (ついでに, 処理の実行前には ConcurrentMark::set_phase() も呼んで... #TODO)
      ---------------------------------------- -}

	  if (G1CollectedHeap::use_parallel_gc_threads()) {
	    G1CollectedHeap::StrongRootsScope srs(g1h);
	    // this is remark, so we'll use up all available threads
	    int active_workers = ParallelGCThreads;
	    set_phase(active_workers, false /* concurrent */);
	
	    CMRemarkTask remarkTask(this);
	    // We will start all available threads, even if we decide that the
	    // active_workers will be fewer. The extra ones will just bail out
	    // immediately.
	    int n_workers = g1h->workers()->total_workers();
	    g1h->set_par_threads(n_workers);
	    g1h->workers()->run_task(&remarkTask);
	    g1h->set_par_threads(0);
	  } else {
	    G1CollectedHeap::StrongRootsScope srs(g1h);
	    // this is remark, so we'll use up all available threads
	    int active_workers = 1;
	    set_phase(active_workers, false /* concurrent */);
	
	    CMRemarkTask remarkTask(this);
	    // We will start all available threads, even if we decide that the
	    // active_workers will be fewer. The extra ones will just bail out
	    // immediately.
	    remarkTask.work(0);
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  SATBMarkQueueSet& satb_mq_set = JavaThread::satb_mark_queue_set();
	  guarantee(satb_mq_set.completed_buffers_num() == 0, "invariant");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  print_stats();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	#if VERIFY_OBJS_PROCESSED
	  if (_scan_obj_cl.objs_processed != ThreadLocalObjQueue::objs_enqueued) {
	    gclog_or_tty->print_cr("Processed = %d, enqueued = %d.",
	                           _scan_obj_cl.objs_processed,
	                           ThreadLocalObjQueue::objs_enqueued);
	    guarantee(_scan_obj_cl.objs_processed ==
	              ThreadLocalObjQueue::objs_enqueued,
	              "Different number of objs processed and enqueued.");
	  }
	#endif
	}
	
```


