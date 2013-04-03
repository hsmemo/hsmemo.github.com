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
void ConcurrentMark::checkpointRootsInitialPost() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap*   g1h = G1CollectedHeap::heap();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // If we force an overflow during remark, the remark operation will
	  // actually abort and we'll restart concurrent marking. If we always
	  // force an oveflow during remark we'll never actually complete the
	  // marking phase. So, we initilize this here, at the start of the
	  // cycle, so that at the remaining overflow number will decrease at
	  // every remark and we'll eventually not need to cause one.
	  force_overflow_stw()->init();
	
  {- -------------------------------------------
  (1) これから concurrent marking 処理を行うので, 
      NoteStartOfMarkHRClosure を使って
      各 HeapRegion の next TAMS に現在の top 位置を記録しておく.
      ---------------------------------------- -}

	  // For each region note start of marking.
	  NoteStartOfMarkHRClosure startcl;
	  g1h->heap_region_iterate(&startcl);
	
  {- -------------------------------------------
  (1) これから GC を行うので, ReferenceProcessor オブジェクトをリセットしておく.
      ---------------------------------------- -}

	  // Start weak-reference discovery.
	  ReferenceProcessor* rp = g1h->ref_processor();
	  rp->verify_no_references_recorded();
	  rp->enable_discovery(); // enable ("weak") refs discovery
	  rp->setup_policy(false); // snapshot the soft ref policy to be used in this cycle
	
  {- -------------------------------------------
  (1) これから concurrent marking 処理を行うので, 
      SATBMarkQueueSet::set_active_all_threads() を呼んで
      write barrier を concurrent marking 時のモード(ポインタを記録するモード)にしておく.
      (See: [here](no2114EV0.html) for details)
      ---------------------------------------- -}

	  SATBMarkQueueSet& satb_mq_set = JavaThread::satb_mark_queue_set();
	  // This is the start of  the marking cycle, we're expected all
	  // threads to have SATB queues with active set to false.
	  satb_mq_set.set_active_all_threads(true, /* new active value */
	                                     false /* expected_active */);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // update_g1_committed() will be called at the end of an evac pause
	  // when marking is on. So, it's also called at the end of the
	  // initial-mark pause to update the heap end, if the heap expands
	  // during it. No need to call it here.
	}
	
```


