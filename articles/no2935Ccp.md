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
  void work(int i) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    double start = os::elapsedTime();
	    FreeRegionList local_cleanup_list("Local Cleanup List");
	    HumongousRegionSet humongous_proxy_set("Local Cleanup Humongous Proxy Set");
	    HRRSCleanupTask hrrs_cleanup_task;
	    G1NoteEndOfConcMarkClosure g1_note_end(_g1h, i, &local_cleanup_list,
	                                           &humongous_proxy_set,
	                                           &hrrs_cleanup_task);

  {- -------------------------------------------
  (1) G1NoteEndOfConcMarkClosure を引数として
      G1CollectedHeap::heap_region_par_iterate_chunked() または G1CollectedHeap::heap_region_iterate() を呼び出し, 
      Cleanup 処理を行う.
  
      (G1CollectedHeap::use_parallel_gc_threads() が true であれば, 
       G1CollectedHeap::heap_region_par_iterate_chunked() を使ってマルチスレッドで処理する.
       そうでなければ, G1CollectedHeap::heap_region_iterate() によるシングルスレッド処理を行う.)
      ---------------------------------------- -}

	    if (G1CollectedHeap::use_parallel_gc_threads()) {
	      _g1h->heap_region_par_iterate_chunked(&g1_note_end, i,
	                                            HeapRegion::NoteEndClaimValue);
	    } else {
	      _g1h->heap_region_iterate(&g1_note_end);
	    }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(g1_note_end.complete(), "Shouldn't have yielded!");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Now update the lists
	    _g1h->update_sets_after_freeing_regions(g1_note_end.freed_bytes(),
	                                            NULL /* free_list */,
	                                            &humongous_proxy_set,
	                                            true /* par */);
	    {
	      MutexLockerEx x(ParGCRareEvent_lock, Mutex::_no_safepoint_check_flag);
	      _max_live_bytes += g1_note_end.max_live_bytes();
	      _freed_bytes += g1_note_end.freed_bytes();
	
	      _cleanup_list->add_as_tail(&local_cleanup_list);
	      assert(local_cleanup_list.is_empty(), "post-condition");
	
	      HeapRegionRemSet::finish_cleanup_task(&hrrs_cleanup_task);
	    }

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    double end = os::elapsedTime();
	    if (G1PrintParCleanupStats) {
	      gclog_or_tty->print("     Worker thread %d [%8.3f..%8.3f = %8.3f ms] "
	                          "claimed %d regions (tot = %8.3f ms, max = %8.3f ms).\n",
	                          i, start, end, (end-start)*1000.0,
	                          g1_note_end.regions_claimed(),
	                          g1_note_end.claimed_region_time_sec()*1000.0,
	                          g1_note_end.max_region_time_sec()*1000.0);
	    }
	  }
	
```


