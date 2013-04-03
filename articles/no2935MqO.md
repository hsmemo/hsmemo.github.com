---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
bool G1CollectedHeap::do_collection(bool explicit_gc,
                                    bool clear_all_soft_refs,
                                    size_t word_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_at_safepoint(true /* should_be_vm_thread */);
	
  {- -------------------------------------------
  (1) もし GC_locker によって GC が禁止されていれば, ここでリターン.
      ---------------------------------------- -}

	  if (GC_locker::check_active_before_gc()) {
	    return false;
	  }
	
  {- -------------------------------------------
  (1) (DTrace のフック点) (JVMTI のフック点)
      (See: SvcGCMarker)
      ---------------------------------------- -}

	  SvcGCMarker sgcm(SvcGCMarker::FULL);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintHeapAtGC) {
	    Universe::print_heap_before_gc();
	  }
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  verify_region_sets_optional();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const bool do_clear_all_soft_refs = clear_all_soft_refs ||
	                           collector_policy()->should_clear_all_soft_refs();
	
	  ClearedAllSoftRefs casr(do_clear_all_soft_refs, collector_policy());
	
  {- -------------------------------------------
  (1) (変数宣言など) (See: IsGCActiveMark)
      ---------------------------------------- -}

	  {
	    IsGCActiveMark x;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    // Timing
	    bool system_gc = (gc_cause() == GCCause::_java_lang_system_gc);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(!system_gc || explicit_gc, "invariant");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    gclog_or_tty->date_stamp(PrintGC && PrintGCDateStamps);
	    TraceCPUTime tcpu(PrintGCDetails, true, gclog_or_tty);
	    TraceTime t(system_gc ? "Full GC (System.gc())" : "Full GC",
	                PrintGC, true, gclog_or_tty);
	
	    TraceCollectorStats tcs(g1mm()->full_collection_counters());
	    TraceMemoryManagerStats tms(true /* fullGC */, gc_cause());
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (処理の開始時間を記録しておく)
      ---------------------------------------- -}

	    double start = os::elapsedTime();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    g1_policy()->record_full_collection_start();
	
  {- -------------------------------------------
  (1) G1CollectedHeap::wait_while_free_regions_coming() を呼んで, 
      Concurrent Mark が見つけた free region があり, しかもまだ処理途中の段階であれば, 
      それらの処理が終わるまで待機しておく.
      ---------------------------------------- -}

	    wait_while_free_regions_coming();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    append_secondary_free_list_if_not_empty_with_lock();
	
	    gc_prologue(true);

  {- -------------------------------------------
  (1) GC を行うことになったので, CollectedHeap::total_collections() の値を増加させておく.
      (See: CollectedHeap::total_collections())
      ---------------------------------------- -}

	    increment_total_collections(true /* full gc */);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    size_t g1h_prev_used = used();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(used() == recalculate_used(), "Should be equal");
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	    if (VerifyBeforeGC && total_collections() >= VerifyGCStartAt) {
	      HandleMark hm;  // Discard invalid handles created during verification
	      gclog_or_tty->print(" VerifyBeforeGC:");
	      prepare_for_verify();
	      Universe::verify(true);
	    }
	
  {- -------------------------------------------
  (1) #ifdef COMPILER2 の場合には, (これから GC を行うので) DerivedPointerTable の値をリセットしておく.
      (See: DerivedPointerTable)
      ---------------------------------------- -}

	    COMPILER2_PRESENT(DerivedPointerTable::clear());
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // We want to discover references, but not process them yet.
	    // This mode is disabled in
	    // instanceRefKlass::process_discovered_references if the
	    // generation does some collection work, or
	    // instanceRefKlass::enqueue_discovered_references if the
	    // generation returns without doing any work.
	    ref_processor()->disable_discovery();
	    ref_processor()->abandon_partial_discovery();
	    ref_processor()->verify_no_references_recorded();
	
	    // Abandon current iterations of concurrent marking and concurrent
	    // refinement, if any are in progress.
	    concurrent_mark()->abort();
	
	    // Make sure we'll choose a new allocation region afterwards.
	    release_mutator_alloc_region();
	    abandon_gc_alloc_regions();
	    g1_rem_set()->cleanupHRRS();

  {- -------------------------------------------
  (1) G1CollectedHeap::tear_down_region_lists() を呼んで, 
      MasterFreeRegionList の中身を空にしておく
      (中身は, 後で G1CollectedHeap::rebuild_region_lists() によって再構築する).
      ---------------------------------------- -}

	    tear_down_region_lists();
	
	    // We may have added regions to the current incremental collection
	    // set between the last GC or pause and now. We need to clear the
	    // incremental collection set and then start rebuilding it afresh
	    // after this full GC.
	    abandon_collection_set(g1_policy()->inc_cset_head());
	    g1_policy()->clear_incremental_cset();
	    g1_policy()->stop_incremental_cset_building();
	
	    if (g1_policy()->in_young_gc_mode()) {
	      empty_young_list();
	      g1_policy()->set_full_young_gcs(true);
	    }
	
	    // See the comment in G1CollectedHeap::ref_processing_init() about
	    // how reference processing currently works in G1.
	
	    // Temporarily make reference _discovery_ single threaded (non-MT).
	    ReferenceProcessorMTDiscoveryMutator rp_disc_ser(ref_processor(), false);
	
	    // Temporarily make refs discovery atomic
	    ReferenceProcessorAtomicMutator rp_disc_atomic(ref_processor(), true);
	
	    // Temporarily clear _is_alive_non_header
	    ReferenceProcessorIsAliveMutator rp_is_alive_null(ref_processor(), NULL);
	
	    ref_processor()->enable_discovery();
	    ref_processor()->setup_policy(do_clear_all_soft_refs);
	
  {- -------------------------------------------
  (1) G1MarkSweep::invoke_at_safepoint() を呼んで, Full GC を実行する.
      ---------------------------------------- -}

	    // Do collection work
	    {
	      HandleMark hm;  // Discard invalid handles created during gc
	      G1MarkSweep::invoke_at_safepoint(ref_processor(), do_clear_all_soft_refs);
	    }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(free_regions() == 0, "we should not have added any free regions");

  {- -------------------------------------------
  (1) G1CollectedHeap::rebuild_region_lists() を呼んで, 
      MasterFreeRegionList の中身を正しく設定する.
      ---------------------------------------- -}

	    rebuild_region_lists();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    _summary_bytes_used = recalculate_used();
	
  {- -------------------------------------------
  (1) 参照オブジェクト(java.lang.ref オブジェクト)に対する後始末を行う.
      (ReferenceProcessor::enqueue_discovered_references() で
      リストに残った参照オブジェクト(= 差し先が死んだため特殊な処理が必要な参照オブジェクト)を pending list に追加する)
      ---------------------------------------- -}

	    ref_processor()->enqueue_discovered_references();
	
  {- -------------------------------------------
  (1) #ifdef COMPILER2 の場合には, GC が終わったので
      見つかった derived pointer の値を修正しておく.
      (See: DerivedPointerTable)
      ---------------------------------------- -}

	    COMPILER2_PRESENT(DerivedPointerTable::update_pointers());
	
  {- -------------------------------------------
  (1) (プロファイル情報の記録)(JMM 用) (See: MemoryUsage)
      及び (JMM のフック点) でもある  (See: LowMemoryDetector)
      ---------------------------------------- -}

	    MemoryService::track_memory_usage();
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	    if (VerifyAfterGC && total_collections() >= VerifyGCStartAt) {
	      HandleMark hm;  // Discard invalid handles created during verification
	      gclog_or_tty->print(" VerifyAfterGC:");
	      prepare_for_verify();
	      Universe::verify(false);
	    }
	    NOT_PRODUCT(ref_processor()->verify_no_references_recorded());
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    reset_gc_time_stamp();
	    // Since everything potentially moved, we will clear all remembered
	    // sets, and clear all cards.  Later we will rebuild remebered
	    // sets. We will also reset the GC time stamps of the regions.
	    PostMCRemSetClearClosure rs_clear(mr_bs());
	    heap_region_iterate(&rs_clear);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Resize the heap if necessary.
	    resize_if_necessary_after_full_collection(explicit_gc ? 0 : word_size);
	
	    if (_cg1r->use_cache()) {
	      _cg1r->clear_and_record_card_counts();
	      _cg1r->clear_hot_cache();
	    }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Rebuild remembered sets of all regions.
	
	    if (G1CollectedHeap::use_parallel_gc_threads()) {
	      ParRebuildRSTask rebuild_rs_task(this);
	      assert(check_heap_region_claim_values(
	             HeapRegion::InitialClaimValue), "sanity check");
	      set_par_threads(workers()->total_workers());
	      workers()->run_task(&rebuild_rs_task);
	      set_par_threads(0);
	      assert(check_heap_region_claim_values(
	             HeapRegion::RebuildRSClaimValue), "sanity check");
	      reset_heap_region_claim_values();
	    } else {
	      RebuildRSOutOfRegionClosure rebuild_rs(this);
	      heap_region_iterate(&rebuild_rs);
	    }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    if (PrintGC) {
	      print_size_transition(gclog_or_tty, g1h_prev_used, used(), capacity());
	    }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    if (true) { // FIXME
	      // Ask the permanent generation to adjust size for full collections
	      perm()->compute_new_size();
	    }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Start a new incremental collection set for the next pause
	    assert(g1_policy()->collection_set() == NULL, "must be");
	    g1_policy()->start_incremental_cset_building();
	
	    // Clear the _cset_fast_test bitmap in anticipation of adding
	    // regions to the incremental collection set for the next
	    // evacuation pause.
	    clear_cset_fast_test();
	
	    init_mutator_alloc_region();
	
	    double end = os::elapsedTime();
	    g1_policy()->record_full_collection_end();
	
	#ifdef TRACESPINNING
	    ParallelTaskTerminator::print_termination_counts();
	#endif
	
	    gc_epilogue(true);
	
	    // Discard all rset updates
	    JavaThread::dirty_card_queue_set().abandon_logs();
	    assert(!G1DeferredRSUpdate
	           || (G1DeferredRSUpdate && (dirty_card_queue_set().completed_buffers_num() == 0)), "Should not be any");
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (g1_policy()->in_young_gc_mode()) {
	    _young_list->reset_sampled_info();
	    // At this point there should be no regions in the
	    // entire heap tagged as young.
	    assert( check_young_list_empty(true /* check_heap */),
	            "young list should be empty at this point");
	  }
	
  {- -------------------------------------------
  (1) (Full GC の完了を待っているスレッドがいるかもしれないので)
      G1CollectedHeap::increment_full_collections_completed() を呼び出して
      待っているスレッドがいれば起床させておく.
      (See: VM_G1IncCollectionPause::doit_epilogue())
      ---------------------------------------- -}

	  // Update the number of full collections that have been completed.
	  increment_full_collections_completed(false /* concurrent */);
	
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  verify_region_sets_optional();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintHeapAtGC) {
	    Universe::print_heap_after_gc();
	  }

  {- -------------------------------------------
  (1) (プロファイル情報の記録) (JMM 用)
      (See: G1MonitoringSupport::update_counters())
      ---------------------------------------- -}

	  g1mm()->update_counters();
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return true;
	}
	
```


