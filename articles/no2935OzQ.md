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
  bool doHeapRegion(HeapRegion* hr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 初回の呼び出し時であれば _start_vtime_sec を初期化しておく
      ---------------------------------------- -}

	    if (!_final && _regions_done == 0)
	      _start_vtime_sec = os::elapsedVTime();
	
  {- -------------------------------------------
  (1) もし対象の HeapRegion が Humongous オブジェクトの途中に当たる場合は
      特にすることはないので, ここでリターン.
      ---------------------------------------- -}

	    if (hr->continuesHumongous()) {
	      // We will ignore these here and process them when their
	      // associated "starts humongous" region is processed (see
	      // set_bit_for_heap_region()). Note that we cannot rely on their
	      // associated "starts humongous" region to have their bit set to
	      // 1 since, due to the region chunking in the parallel region
	      // iteration, a "continues humongous" region might be visited
	      // before its associated "starts humongous".
	      return false;
	    }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    HeapWord* nextTop = hr->next_top_at_mark_start();
	    HeapWord* start   = hr->top_at_conc_mark_count();
	    assert(hr->bottom() <= start && start <= hr->end() &&
	           hr->bottom() <= nextTop && nextTop <= hr->end() &&
	           start <= nextTop,
	           "Preconditions.");
	    // Otherwise, record the number of word's we'll examine.
	    size_t words_done = (nextTop - start);

  {- -------------------------------------------
  (1) CMBitMapRO::getNextMarkedWordAddress() を呼んで, 
      ---------------------------------------- -}

	    // Find the first marked object at or after "start".
	    start = _bm->getNextMarkedWordAddress(start, nextTop);
	    size_t marked_bytes = 0;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    // Below, the term "card num" means the result of shifting an address
	    // by the card shift -- address 0 corresponds to card number 0.  One
	    // must subtract the card num of the bottom of the heap to obtain a
	    // card table index.
	    // The first card num of the sequence of live cards currently being
	    // constructed.  -1 ==> no sequence.
	    intptr_t start_card_num = -1;
	    // The last card num of the sequence of live cards currently being
	    // constructed.  -1 ==> no sequence.
	    intptr_t last_card_num = -1;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    while (start < nextTop) {
	      if (_yield && _cm->do_yield_check()) {
	        // We yielded.  It might be for a full collection, in which case
	        // all bets are off; terminate the traversal.
	        if (_cm->has_aborted()) {
	          _changed = false;
	          return true;
	        } else {
	          // Otherwise, it might be a collection pause, and the region
	          // we're looking at might be in the collection set.  We'll
	          // abandon this region.
	          return false;
	        }
	      }
	      oop obj = oop(start);
	      int obj_sz = obj->size();
	      // The card num of the start of the current object.
	      intptr_t obj_card_num =
	        intptr_t(uintptr_t(start) >> CardTableModRefBS::card_shift);
	
	      HeapWord* obj_last = start + obj_sz - 1;
	      intptr_t obj_last_card_num =
	        intptr_t(uintptr_t(obj_last) >> CardTableModRefBS::card_shift);
	
	      if (obj_card_num != last_card_num) {
	        if (start_card_num == -1) {
	          assert(last_card_num == -1, "Both or neither.");
	          start_card_num = obj_card_num;
	        } else {
	          assert(last_card_num != -1, "Both or neither.");
	          assert(obj_card_num >= last_card_num, "Inv");
	          if ((obj_card_num - last_card_num) > 1) {
	            // Mark the last run, and start a new one.
	            mark_card_num_range(start_card_num, last_card_num);
	            start_card_num = obj_card_num;
	          }
	        }
	#if CARD_BM_TEST_MODE
	        /*
	        gclog_or_tty->print_cr("Setting bits from %d/%d.",
	                               obj_card_num - _bottom_card_num,
	                               obj_last_card_num - _bottom_card_num);
	        */
	        for (intptr_t j = obj_card_num; j <= obj_last_card_num; j++) {
	          _card_bm->par_at_put(j - _bottom_card_num, 1);
	        }
	#endif
	      }
	      // In any case, we set the last card num.
	      last_card_num = obj_last_card_num;
	
	      marked_bytes += (size_t)obj_sz * HeapWordSize;
	      // Find the next marked object after this one.
	      start = _bm->getNextMarkedWordAddress(start + 1, nextTop);
	      _changed = true;
	    }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Handle the last range, if any.
	    if (start_card_num != -1)
	      mark_card_num_range(start_card_num, last_card_num);
	    if (_final) {
	      // Mark the allocated-since-marking portion...
	      HeapWord* tp = hr->top();
	      if (nextTop < tp) {
	        start_card_num =
	          intptr_t(uintptr_t(nextTop) >> CardTableModRefBS::card_shift);
	        last_card_num =
	          intptr_t(uintptr_t(tp) >> CardTableModRefBS::card_shift);
	        mark_card_num_range(start_card_num, last_card_num);
	        // This definitely means the region has live objects.
	        set_bit_for_region(hr);
	      }
	    }
	
	    hr->add_to_marked_bytes(marked_bytes);
	    // Update the live region bitmap.
	    if (marked_bytes > 0) {
	      set_bit_for_region(hr);
	    }
	    hr->set_top_at_conc_mark_count(nextTop);
	    _tot_live += hr->next_live_bytes();
	    _tot_used += hr->used();
	    _words_done = words_done;
	
  {- -------------------------------------------
  (1) Concurrent 処理中に呼ばれた場合は (= _final フィールドが false の場合は), 
      Concurrent marking によるオーバーヘッドを平準化するため, 
      適当な時間だけ os::sleep() で待機しておく
    
      (正確には, 呼び出された回数を _regions_done に記録しており, 10 回呼び出されると 1回待機する.
       待機時間は, 前回の待機処理以降の処理時間(以下の elapsed_vtime_sec)に比例.
       ただし, 処理時間(elapsed_vtime_sec)があまりに短い場合(0.01 sec未満)には, 待機処理は(もう10回分先まで)延期する)
      ---------------------------------------- -}

	    if (!_final) {
	      ++_regions_done;
	      if (_regions_done % 10 == 0) {
	        double end_vtime_sec = os::elapsedVTime();
	        double elapsed_vtime_sec = end_vtime_sec - _start_vtime_sec;
	        if (elapsed_vtime_sec > (10.0 / 1000.0)) {
	          jlong sleep_time_ms =
	            (jlong) (elapsed_vtime_sec * _cm->cleanup_sleep_factor() * 1000.0);
	          os::sleep(Thread::current(), sleep_time_ms, false);
	          _start_vtime_sec = end_vtime_sec;
	        }
	      }
	    }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	    return false;
	  }
	
```


