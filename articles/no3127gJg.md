---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp

### 名前(function name)
```
bool G1RemSet::concurrentRefineOneCard_impl(jbyte* card_ptr, int worker_i,
                                                   bool check_for_refs_into_cset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)??
      (dirtyRegion は, 処理対象の card に対応するアドレス範囲を示す)
      ---------------------------------------- -}

	  // Construct the region representing the card.
	  HeapWord* start = _ct_bs->addr_for(card_ptr);
	  // And find the region containing it.
	  HeapRegion* r = _g1->heap_region_containing(start);
	  assert(r != NULL, "unexpected null");
	
	  HeapWord* end   = _ct_bs->addr_for(card_ptr + 1);
	  MemRegion dirtyRegion(start, end);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (CARD_REPEAT_HISTO マクロ定数を非 0 に #define した場合にのみ実行)
      ---------------------------------------- -}

	#if CARD_REPEAT_HISTO
	  init_ct_freq_table(_g1->max_capacity());
	  ct_freq_note_card(_ct_bs->index_for(start));
	#endif
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!check_for_refs_into_cset || _cset_rs_update_cl[worker_i] != NULL, "sanity");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  UpdateRSOrPushRefOopClosure update_rs_oop_cl(_g1,
	                                               _g1->g1_rem_set(),
	                                               _cset_rs_update_cl[worker_i],
	                                               check_for_refs_into_cset,
	                                               worker_i);

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  update_rs_oop_cl.set_from(r);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  TriggerClosure trigger_cl;
	  FilterIntoCSClosure into_cs_cl(NULL, _g1, &trigger_cl);
	  InvokeIfNotTriggeredClosure invoke_cl(&trigger_cl, &into_cs_cl);
	  Mux2Closure mux(&invoke_cl, &update_rs_oop_cl);
	
	  FilterOutOfRegionClosure filter_then_update_rs_oop_cl(r,
	                        (check_for_refs_into_cset ?
	                                (OopClosure*)&mux :
	                                (OopClosure*)&update_rs_oop_cl));
	
  {- -------------------------------------------
  (1) HeapRegion::oops_on_card_seq_iterate_careful() を呼んで,
      実際の処理を行う.
      ---------------------------------------- -}

	  // The region for the current card may be a young region. The
	  // current card may have been a card that was evicted from the
	  // card cache. When the card was inserted into the cache, we had
	  // determined that its region was non-young. While in the cache,
	  // the region may have been freed during a cleanup pause, reallocated
	  // and tagged as young.
	  //
	  // We wish to filter out cards for such a region but the current
	  // thread, if we're running concurrently, may "see" the young type
	  // change at any time (so an earlier "is_young" check may pass or
	  // fail arbitrarily). We tell the iteration code to perform this
	  // filtering when it has been determined that there has been an actual
	  // allocation in this region and making it safe to check the young type.
	  bool filter_young = true;
	
	  HeapWord* stop_point =
	    r->oops_on_card_seq_iterate_careful(dirtyRegion,
	                                        &filter_then_update_rs_oop_cl,
	                                        filter_young,
	                                        card_ptr);
	
  {- -------------------------------------------
  (1) 途中で unparseable なオブジェクトが見つかった場合は, 再度 enqueue しておく.
  
      (逆に成功したら, out_of_histo.add_entry() して統計情報をためたり,
      _conc_refine_cards++ をインクリメントしている)
      ---------------------------------------- -}

	  // If stop_point is non-null, then we encountered an unallocated region
	  // (perhaps the unfilled portion of a TLAB.)  For now, we'll dirty the
	  // card and re-enqueue: if we put off the card until a GC pause, then the
	  // unallocated portion will be filled in.  Alternatively, we might try
	  // the full complexity of the technique used in "regular" precleaning.
	  if (stop_point != NULL) {
	    // The card might have gotten re-dirtied and re-enqueued while we
	    // worked.  (In fact, it's pretty likely.)
	    if (*card_ptr != CardTableModRefBS::dirty_card_val()) {
	      *card_ptr = CardTableModRefBS::dirty_card_val();
	      MutexLockerEx x(Shared_DirtyCardQ_lock,
	                      Mutex::_no_safepoint_check_flag);
	      DirtyCardQueue* sdcq =
	        JavaThread::dirty_card_queue_set().shared_dirty_card_queue();
	      sdcq->enqueue(card_ptr);
	    }
	  } else {
	    out_of_histo.add_entry(filter_then_update_rs_oop_cl.out_of_region());
	    _conc_refine_cards++;
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return trigger_cl.value();
	}
	
```


