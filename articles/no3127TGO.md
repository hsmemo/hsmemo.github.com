---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp

### 名前(function name)
```
void OtherRegionsTable::add_reference(OopOrNarrowOopStar from, int tid) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  size_t cur_hrs_ind = hr()->hrs_index();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if HRRS_VERBOSE
	  gclog_or_tty->print_cr("ORT::add_reference_work(" PTR_FORMAT "->" PTR_FORMAT ").",
	                                                  from,
	                                                  UseCompressedOops
	                                                  ? oopDesc::load_decode_heap_oop((narrowOop*)from)
	                                                  : oopDesc::load_decode_heap_oop((oop*)from));
	#endif
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int from_card = (int)(uintptr_t(from) >> CardTableModRefBS::card_shift);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if HRRS_VERBOSE
	  gclog_or_tty->print_cr("Table for [" PTR_FORMAT "...): card %d (cache = %d)",
	                hr()->bottom(), from_card,
	                _from_card_cache[tid][cur_hrs_ind]);
	#endif
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#define COUNT_CACHE 0
	#if COUNT_CACHE
	  jint p = Atomic::add(1, &_cache_probes);
	  if ((p % 10000) == 0) {
	    jint hits = _cache_hits;
	    gclog_or_tty->print_cr("%d/%d = %5.2f%% RS cache hits.",
	                  _cache_hits, p, 100.0* (float)hits/(float)p);
	  }
	#endif

  {- -------------------------------------------
  (1) ...#TODO の場合はここでリターン. 
      (ついでに(トレース出力)も出している)
  
      (そうでない場合は _from_card_cache の値をセットしている. <= これはなに?? #TODO)
      ---------------------------------------- -}

	  if (from_card == _from_card_cache[tid][cur_hrs_ind]) {
	#if HRRS_VERBOSE
	    gclog_or_tty->print_cr("  from-card cache hit.");
	#endif
	#if COUNT_CACHE
	    Atomic::inc(&_cache_hits);
	#endif
	    assert(contains_reference(from), "We just added it!");
	    return;
	  } else {
	    _from_card_cache[tid][cur_hrs_ind] = from_card;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Note that this may be a continued H region.
	  HeapRegion* from_hr = _g1h->heap_region_containing_raw(from);
	  RegionIdx_t from_hrs_ind = (RegionIdx_t) from_hr->hrs_index();
	
  {- -------------------------------------------
  (1) 既に _coarse_map の方に登録されている場合は, ここでリターン.
      (ついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  // If the region is already coarsened, return.
	  if (_coarse_map.at(from_hrs_ind)) {
	#if HRRS_VERBOSE
	    gclog_or_tty->print_cr("  coarse map hit.");
	#endif
	    assert(contains_reference(from), "We just added it!");
	    return;
	  }
	
  {- -------------------------------------------
  (1) find_region_table() を呼んで, 対応する PosParPRT を取得する.
      (なお, もしまだ作成されてなければ以下の if ブロック内で作成する)
      ---------------------------------------- -}

	  // Otherwise find a per-region table to add it to.
	  size_t ind = from_hrs_ind & _mod_max_fine_entries_mask;
	  PosParPRT* prt = find_region_table(ind, from_hr);
	  if (prt == NULL) {

    {- -------------------------------------------
  (1.1) DCL idiom
        ---------------------------------------- -}

	    MutexLockerEx x(&_m, Mutex::_no_safepoint_check_flag);
	    // Confirm that it's really not there...
	    prt = find_region_table(ind, from_hr);
	    if (prt == NULL) {

    {- -------------------------------------------
  (1.1) (以下はまだ作成されてない場合のパス)
        ---------------------------------------- -}
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	      uintptr_t from_hr_bot_card_index =
	        uintptr_t(from_hr->bottom())
	          >> CardTableModRefBS::card_shift;
	      CardIdx_t card_index = from_card - from_hr_bot_card_index;
	      assert(0 <= card_index && card_index < HeapRegion::CardsPerRegion,
	             "Must be in range.");

    {- -------------------------------------------
  (1.1) G1HRRSUseSparseTable オプションが指定されていれば,
        まずは SparsePRT::add_card() を呼んで SparsePRT 内への登録を試みる.
        成功すれば, ここでリターン.
        ---------------------------------------- -}

	      if (G1HRRSUseSparseTable &&
	          _sparse_table.add_card(from_hrs_ind, card_index)) {
	        if (G1RecordHRRSOops) {
	          HeapRegionRemSet::record(hr(), from);
	#if HRRS_VERBOSE
	          gclog_or_tty->print("   Added card " PTR_FORMAT " to region "
	                              "[" PTR_FORMAT "...) for ref " PTR_FORMAT ".\n",
	                              align_size_down(uintptr_t(from),
	                                              CardTableModRefBS::card_size),
	                              hr()->bottom(), from);
	#endif
	        }
	#if HRRS_VERBOSE
	        gclog_or_tty->print_cr("   added card to sparse table.");
	#endif
	        assert(contains_reference_locked(from), "We just added it!");
	        return;
	      } else {
	#if HRRS_VERBOSE
	        gclog_or_tty->print_cr("   [tid %d] sparse table entry "
	                      "overflow(f: %d, t: %d)",
	                      tid, from_hrs_ind, cur_hrs_ind);
	#endif
	      }
	
    {- -------------------------------------------
  (1.1) 今回の要素用の PosParPRT を確保する.
        (この処理には2通りある.
        もうエントリがいっぱいであれば, OtherRegionsTable::delete_region_table() で _coarse_map に追い出してエントリを確保する.
        そうでなければ, PosParPRT::alloc() を使用する)
        ---------------------------------------- -}

	      if (_n_fine_entries == _max_fine_entries) {
	        prt = delete_region_table();
	      } else {
	        prt = PosParPRT::alloc(from_hr);
	      }

    {- -------------------------------------------
  (1.1) 確保した PosParPRT を初期化.
        ---------------------------------------- -}

	      prt->init(from_hr);
	
    {- -------------------------------------------
  (1.1) 確保した PosParPRT を, _fine_grain_regions 内の該当するバケットの先頭につなぐ.
        (ついでに _n_fine_entries もインクリメントしておく)
        ---------------------------------------- -}

	      PosParPRT* first_prt = _fine_grain_regions[ind];
	      prt->set_next(first_prt);  // XXX Maybe move to init?
	      _fine_grain_regions[ind] = prt;
	      _n_fine_entries++;
	
    {- -------------------------------------------
  (1.1) PosParPRT が確保できたので,
        G1HRRSUseSparseTable オプションが指定されている場合には
        対応する SparsePRTEntry から確保した PosParPRT に情報を移しておく.
        移し終わったら SparsePRTEntry の方は開放.
        ---------------------------------------- -}

	      if (G1HRRSUseSparseTable) {
	        // Transfer from sparse to fine-grain.
	        SparsePRTEntry *sprt_entry = _sparse_table.get_entry(from_hrs_ind);
	        assert(sprt_entry != NULL, "There should have been an entry");
	        for (int i = 0; i < SparsePRTEntry::cards_num(); i++) {
	          CardIdx_t c = sprt_entry->card(i);
	          if (c != SparsePRTEntry::NullEntry) {
	            prt->add_card(c);
	          }
	        }
	        // Now we can delete the sparse entry.
	        bool res = _sparse_table.delete_entry(from_hrs_ind);
	        assert(res, "It should have been there.");
	      }
	    }

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    assert(prt != NULL && prt->hr() == from_hr, "consequence");
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Note that we can't assert "prt->hr() == from_hr", because of the
	  // possibility of concurrent reuse.  But see head comment of
	  // OtherRegionsTable for why this is OK.
	  assert(prt != NULL, "Inv");
	
  {- -------------------------------------------
  (1) PosParPRT::add_reference(OopOrNarrowOopStar from, int tid)で Remembered Set に情報を記録する.
      (なお expand が必要であれば par_expand() を呼んだあとで追加する.)
      ---------------------------------------- -}

	  if (prt->should_expand(tid)) {
	    MutexLockerEx x(&_m, Mutex::_no_safepoint_check_flag);
	    HeapRegion* prt_hr = prt->hr();
	    if (prt_hr == from_hr) {
	      // Make sure the table still corresponds to the same region
	      prt->par_expand();
	      prt->add_reference(from, tid);
	    }
	    // else: The table has been concurrently coarsened, evicted, and
	    // the table data structure re-used for another table. So, we
	    // don't need to add the reference any more given that the table
	    // has been coarsened and the whole region will be scanned anyway.
	  } else {
	    prt->add_reference(from, tid);
	  }

  {- -------------------------------------------
  (1) (トレース出力)??
      ---------------------------------------- -}

	  if (G1RecordHRRSOops) {
	    HeapRegionRemSet::record(hr(), from);
	#if HRRS_VERBOSE
	    gclog_or_tty->print("Added card " PTR_FORMAT " to region "
	                        "[" PTR_FORMAT "...) for ref " PTR_FORMAT ".\n",
	                        align_size_down(uintptr_t(from),
	                                        CardTableModRefBS::card_size),
	                        hr()->bottom(), from);
	#endif
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(contains_reference(from), "We just added it!");
	}
	
```


