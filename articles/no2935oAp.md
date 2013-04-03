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
void ConcurrentMark::cleanup() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // world is stopped at this checkpoint
	  assert(SafepointSynchronize::is_at_safepoint(),
	         "world should be stopped");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	
  {- -------------------------------------------
  (1) 並行して Full GC が実行されてしまった場合は
      (= ConcurrentMark::has_aborted() が true の場合は), 
      これ以上 marking 処理を続けても結果は不正なので, ここでリターン.
    
      (ついでに, G1CollectedHeap::set_marking_complete() を呼んで 
       Concurrent Mark 処理が終了したことが分かるようにしておく.)
      ---------------------------------------- -}

	  // If a full collection has happened, we shouldn't do this.
	  if (has_aborted()) {
	    g1h->set_marking_complete(); // So bitmap clearing isn't confused
	    return;
	  }
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  g1h->verify_region_sets_optional();
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  if (VerifyDuringGC) {
	    HandleMark hm;  // handle scope
	    gclog_or_tty->print(" VerifyDuringGC:(before)");
	    Universe::heap()->prepare_for_verify();
	    Universe::verify(/* allow dirty  */ true,
	                     /* silent       */ false,
	                     /* prev marking */ true);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectorPolicy* g1p = G1CollectedHeap::heap()->g1_policy();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1p->record_concurrent_mark_cleanup_start();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (処理の開始時間を記録しておく)
      ---------------------------------------- -}

	  double start = os::elapsedTime();
	
  {- -------------------------------------------
  (1) HeapRegionRemSet::reset_for_cleanup_tasks() を呼んで, Remembered Set をリセットしておく.
      ---------------------------------------- -}

	  HeapRegionRemSet::reset_for_cleanup_tasks();
	
  {- -------------------------------------------
  (1) G1ParFinalCountTask::work() を呼んで, Live Data Counting 処理を行う.
  
      (なお, G1CollectedHeap::use_parallel_gc_threads() が true の場合は, 
       WorkGang::run_task() 経由で呼び出す.
       これにより, 処理がマルチスレッドで実行される)
      ---------------------------------------- -}

	  // Do counting once more with the world stopped for good measure.
	  G1ParFinalCountTask g1_par_count_task(g1h, nextMarkBitMap(),
	                                        &_region_bm, &_card_bm);
	  if (G1CollectedHeap::use_parallel_gc_threads()) {
	    assert(g1h->check_heap_region_claim_values(
	                                               HeapRegion::InitialClaimValue),
	           "sanity check");
	
	    int n_workers = g1h->workers()->total_workers();
	    g1h->set_par_threads(n_workers);
	    g1h->workers()->run_task(&g1_par_count_task);
	    g1h->set_par_threads(0);
	
	    assert(g1h->check_heap_region_claim_values(
	                                             HeapRegion::FinalCountClaimValue),
	           "sanity check");
	  } else {
	    g1_par_count_task.work(0);
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  size_t known_garbage_bytes =
	    g1_par_count_task.used_bytes() - g1_par_count_task.live_bytes();
	#if 0
	  gclog_or_tty->print_cr("used %1.2lf, live %1.2lf, garbage %1.2lf",
	                         (double) g1_par_count_task.used_bytes() / (double) (1024 * 1024),
	                         (double) g1_par_count_task.live_bytes() / (double) (1024 * 1024),
	                         (double) known_garbage_bytes / (double) (1024 * 1024));
	#endif // 0
	  g1p->set_known_garbage_bytes(known_garbage_bytes);
	
	  size_t start_used_bytes = g1h->used();
	  _at_least_one_mark_complete = true;

  {- -------------------------------------------
  (1) G1CollectedHeap::set_marking_complete() を呼んで 
      Concurrent Mark 処理が終了したことが分かるようにしておく.
      ---------------------------------------- -}

	  g1h->set_marking_complete();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  double count_end = os::elapsedTime();
	  double this_final_counting_time = (count_end - start);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (G1PrintParCleanupStats) {
	    gclog_or_tty->print_cr("Cleanup:");
	    gclog_or_tty->print_cr("  Finalize counting: %8.3f ms",
	                           this_final_counting_time*1000.0);
	  }

  {- -------------------------------------------
  (1) (トレース出力用の処理)
  
      (_total_counting_time は現在は以下のパスでのみ参照されている.
        
       before_exit()
       -> G1CollectedHeap::print_tracing_info()
          -> ConcurrentMark::print_summary_info() (G1SummarizeConcMark オプション指定時のみ))
      ---------------------------------------- -}

	  _total_counting_time += this_final_counting_time;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (G1PrintRegionLivenessInfo) {
	    G1PrintRegionLivenessInfoClosure cl(gclog_or_tty, "Post-Marking");
	    _g1h->heap_region_iterate(&cl);
	  }
	
  {- -------------------------------------------
  (1) ConcurrentMark::swapMarkBitMaps() を呼んで, 
      next marking bitmap と prev marking bitmap を入れ替える.
      ---------------------------------------- -}

	  // Install newly created mark bitMap as "prev".
	  swapMarkBitMaps();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1h->reset_gc_time_stamp();
	
  {- -------------------------------------------
  (1) G1ParNoteEndTask::work() を呼んで, Cleanup 処理を行う.
      (なお, G1CollectedHeap::use_parallel_gc_threads() が true の場合は, 
       WorkGang::run_task() 経由で呼び出す.
       これにより, 処理がマルチスレッドで実行される)
  
      (なお, Cleanup 処理中に空の HeapRegion が見つかった場合は
       (= ConcurrentMark::cleanup_list_is_empty() が true の場合は), 
       G1CollectedHeap::set_free_regions_coming() を呼んで, そのことを記録しておく.
       これにより, ConcurrentMarkThread::run() 内でそれらの HeapRegion が処理されるようになる.
       (See: ConcurrentMarkThread::run(), G1CollectedHeap::wait_while_free_regions_coming()
             G1CollectedHeap::new_region_try_secondary_free_list()))
  
      (なお, 処理の前後で os::elapsedTime() で時間を計測し
       処理に掛かった時間を計算しているが, 
       これは直後に行われるトレース出力用の処理)
      ---------------------------------------- -}

	  // Note end of marking in all heap regions.
	  double note_end_start = os::elapsedTime();
	  G1ParNoteEndTask g1_par_note_end_task(g1h, &_cleanup_list);
	  if (G1CollectedHeap::use_parallel_gc_threads()) {
	    int n_workers = g1h->workers()->total_workers();
	    g1h->set_par_threads(n_workers);
	    g1h->workers()->run_task(&g1_par_note_end_task);
	    g1h->set_par_threads(0);
	
	    assert(g1h->check_heap_region_claim_values(HeapRegion::NoteEndClaimValue),
	           "sanity check");
	  } else {
	    g1_par_note_end_task.work(0);
	  }
	
	  if (!cleanup_list_is_empty()) {
	    // The cleanup list is not empty, so we'll have to process it
	    // concurrently. Notify anyone else that might be wanting free
	    // regions that there will be more free regions coming soon.
	    g1h->set_free_regions_coming();
	  }
	  double note_end_end = os::elapsedTime();

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (G1PrintParCleanupStats) {
	    gclog_or_tty->print_cr("  note end of marking: %8.3f ms.",
	                           (note_end_end - note_end_start)*1000.0);
	  }
	
	
  {- -------------------------------------------
  (1) G1ParScrubRemSetTask::work() を呼んで, 
      生きているオブジェクトがいない HeapRegion について, 
      そこに付いている Remembered Set 用のデータ構造を解放する.
      (なお, G1CollectedHeap::use_parallel_gc_threads() が true の場合は, 
       WorkGang::run_task() 経由で呼び出す.
       これにより, 処理がマルチスレッドで実行される)
  
      (ただし, develop オプションである G1ScrubRemSets が 
       false に設定されている場合には, この処理は省略する)
      ---------------------------------------- -}

	  // call below, since it affects the metric by which we sort the heap
	  // regions.
	  if (G1ScrubRemSets) {
	    double rs_scrub_start = os::elapsedTime();
	    G1ParScrubRemSetTask g1_par_scrub_rs_task(g1h, &_region_bm, &_card_bm);
	    if (G1CollectedHeap::use_parallel_gc_threads()) {
	      int n_workers = g1h->workers()->total_workers();
	      g1h->set_par_threads(n_workers);
	      g1h->workers()->run_task(&g1_par_scrub_rs_task);
	      g1h->set_par_threads(0);
	
	      assert(g1h->check_heap_region_claim_values(
	                                            HeapRegion::ScrubRemSetClaimValue),
	             "sanity check");
	    } else {
	      g1_par_scrub_rs_task.work(0);
	    }
	
	    double rs_scrub_end = os::elapsedTime();
	    double this_rs_scrub_time = (rs_scrub_end - rs_scrub_start);
	    _total_rs_scrub_time += this_rs_scrub_time;
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // this will also free any regions totally full of garbage objects,
	  // and sort the regions.
	  g1h->g1_policy()->record_concurrent_mark_cleanup_end(
	                        g1_par_note_end_task.freed_bytes(),
	                        g1_par_note_end_task.max_live_bytes());
	
  {- -------------------------------------------
  (1) (トレース出力用の処理)
  
      (_cleanup_times は現在は以下のパスでのみ参照されている.
        
       before_exit()
       -> G1CollectedHeap::print_tracing_info()
          -> ConcurrentMark::print_summary_info() (G1SummarizeConcMark オプション指定時のみ))
      ---------------------------------------- -}

	  // Statistics.
	  double end = os::elapsedTime();
	  _cleanup_times.add((end - start) * 1000.0);
	
  {- -------------------------------------------
  (1) (トレース出力)??
      ---------------------------------------- -}

	  // G1CollectedHeap::heap()->print();
	  // gclog_or_tty->print_cr("HEAP GC TIME STAMP : %d",
	  // G1CollectedHeap::heap()->get_gc_time_stamp());
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintGC || PrintGCDetails) {
	    g1h->print_size_transition(gclog_or_tty,
	                               start_used_bytes,
	                               g1h->used(),
	                               g1h->capacity());
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  size_t cleaned_up_bytes = start_used_bytes - g1h->used();
	  g1p->decrease_known_garbage_bytes(cleaned_up_bytes);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // We need to make this be a "collection" so any collection pause that
	  // races with it goes around and waits for completeCleanup to finish.
	  g1h->increment_total_collections();
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  if (VerifyDuringGC) {
	    HandleMark hm;  // handle scope
	    gclog_or_tty->print(" VerifyDuringGC:(after)");
	    Universe::heap()->prepare_for_verify();
	    Universe::verify(/* allow dirty  */ true,
	                     /* silent       */ false,
	                     /* prev marking */ true);
	  }
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  g1h->verify_region_sets_optional();
	}
	
```


