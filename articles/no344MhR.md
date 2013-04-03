---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp


### 本体部(body)
```
	HeapWord* G1CollectedHeap::attempt_allocation_slow(size_t word_size,
	                                           unsigned int *gc_count_before_ret) {

  {- -------------------------------------------
  (1) (G1CollectedHeap::attempt_allocation_humongous() 中のコメントも参照, とのこと)
      ---------------------------------------- -}

	  // Make sure you read the note in attempt_allocation_humongous().
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_heap_not_locked_and_not_at_safepoint();
	  assert(!isHumongous(word_size), "attempt_allocation_slow() should not "
	         "be called for humongous allocation requests");
	
  {- -------------------------------------------
  (1) 以下, メモリ確保が成功するか, あるいは GC を実行したが失敗するまで, 以下のループで処理を続ける
      (以下のループを抜けるのは return する時だけ)
      (GC を実行したが失敗したケースのみ NULL が返される)
      ---------------------------------------- -}

	  // We should only get here after the first-level allocation attempt
	  // (attempt_allocation()) failed to allocate.
	
	  // We will loop until a) we manage to successfully perform the
	  // allocation or b) we successfully schedule a collection which
	  // fails to perform the allocation. b) is the only case when we'll
	  // return NULL.
	  HeapWord* result = NULL;
	  for (int try_count = 1; /* we'll return */; try_count += 1) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    bool should_try_gc;
	    unsigned int gc_count_before;
	
    {- -------------------------------------------
  (1.1) (以下, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    {
	      MutexLockerEx x(Heap_lock);
	
    {- -------------------------------------------
  (1.1) G1AllocRegion::attempt_allocation_locked() でメモリ確保を試みる.
        確保に成功したら, ここでリターン.
        ---------------------------------------- -}

	      result = _mutator_alloc_region.attempt_allocation_locked(word_size,
	                                                      false /* bot_updates */);
	      if (result != NULL) {
	        return result;
	      }
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      // If we reach here, attempt_allocation_locked() above failed to
	      // allocate a new region. So the mutator alloc region should be NULL.
	      assert(_mutator_alloc_region.get() == NULL, "only way to get here");
	
    {- -------------------------------------------
  (1.1) もし, 現在 GC が禁止されており(= GC_locker が active で), かつ 
        既に誰かが GC のエントリ処理を実行して GC の必要性を GC_locker に通知している場合, ...#TODO
        ---------------------------------------- -}

	      if (GC_locker::is_active_and_needs_gc()) {
	        if (g1_policy()->can_expand_young_list()) {
	          result = _mutator_alloc_region.attempt_allocation_force(word_size,
	                                                      false /* bot_updates */);
	          if (result != NULL) {
	            return result;
	          }
	        }
	        should_try_gc = false;

    {- -------------------------------------------
  (1.1) もし上の条件が成り立たなければ (= GC_locker::is_active_and_needs_gc() が false ならば), 
        GC 処理に備えて現在の total_collections() の値を取得し, 
        (See: total_collections())
        そのまま以下の GC 処理へと進む. 
        (ここで should_try_gc を true にしているため, この下で GC 処理のパスに入る)
        ---------------------------------------- -}

	      } else {
	        // Read the GC count while still holding the Heap_lock.
	        gc_count_before = SharedHeap::heap()->total_collections();
	        should_try_gc = true;
	      }

    {- -------------------------------------------
  (1.1) (ここまでが, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    }
	
    {- -------------------------------------------
  (1.1) もし GC_locker::is_active_and_needs_gc() が false だった場合には, 
        G1CollectedHeap::do_collection_pause() で GC を実行して確保を試みる.
        * 確保に成功すれば, 確保結果をリターン.
        * GC 要求は実行されたが確保に失敗した場合には, NULL をリターン
        * (他スレッドが同時に出した GC 要求のせいで)
          GC 要求が実行されなかった場合には, このままフォールスルー.
        ---------------------------------------- -}

	    if (should_try_gc) {
	      bool succeeded;
	      result = do_collection_pause(word_size, gc_count_before, &succeeded);

      {- -------------------------------------------
  (1.1.1) 確保に成功したケース
          ---------------------------------------- -}

	      if (result != NULL) {
	        assert(succeeded, "only way to get back a non-NULL result");
	        return result;
	      }
	
      {- -------------------------------------------
  (1.1.1) GC 要求は実行されたが, 確保に失敗したケース
          ---------------------------------------- -}

	      if (succeeded) {
	        // If we get here we successfully scheduled a collection which
	        // failed to allocate. No point in trying to allocate
	        // further. We'll just return NULL.
	        MutexLockerEx x(Heap_lock);
	        *gc_count_before_ret = SharedHeap::heap()->total_collections();
	        return NULL;
	      }

    {- -------------------------------------------
  (1.1) もし GC_locker::is_active_and_needs_gc() が true だった場合には, 
        GC_locker のロックが解けて GC が実行可能になるまでここで待たせる.
        ---------------------------------------- -}

	    } else {
	      GC_locker::stall_until_clear();
	    }
	
    {- -------------------------------------------
  (1.1) G1AllocRegion::attempt_allocation() でメモリ確保を試みる.
        確保に成功したら, ここでリターン.
  
        (なおここに辿り着くのは, GC_locker のせいで GC が実行できなかった場合か, 
        他スレッドが同時に出した GC 要求のせいで自分が要求した GC が実行されなかった場合.
        どちらにせよ, 他のスレッドが GC を実行して空き領域ができている可能性がある.)
        ---------------------------------------- -}

	    // We can reach here if we were unsuccessul in scheduling a
	    // collection (because another thread beat us to it) or if we were
	    // stalled due to the GC locker. In either can we should retry the
	    // allocation attempt in case another thread successfully
	    // performed a collection and reclaimed enough space. We do the
	    // first attempt (without holding the Heap_lock) here and the
	    // follow-on attempt will be at the start of the next loop
	    // iteration (after taking the Heap_lock).
	    result = _mutator_alloc_region.attempt_allocation(word_size,
	                                                      false /* bot_updates */);
	    if (result != NULL ){
	      return result;
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // Give a warning if we seem to be looping forever.
	    if ((QueuedAllocationWarningCount > 0) &&
	        (try_count % QueuedAllocationWarningCount == 0)) {
	      warning("G1CollectedHeap::attempt_allocation_slow() "
	              "retries %d times", try_count);
	    }
	  }
	
  {- -------------------------------------------
  (1) (以下は決して到達しないパス)
      ---------------------------------------- -}

	  ShouldNotReachHere();
	  return NULL;
	}
	
```


