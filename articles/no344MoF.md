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
HeapWord* G1CollectedHeap::attempt_allocation_humongous(size_t word_size,
                                          unsigned int * gc_count_before_ret) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, 処理自体は G1CollectedHeap::attempt_allocation_slow() に似ているとのこと.
      ただし違う点もそれなりにあるので, 一つにまとめると "if humongous {...}" が大量に存在する
      見通しの悪い関数ができそう(というか以前は実際に1つにまとめられていたがバグの温床になった), 
      という理由で2つに分けられているらしい.)
      ---------------------------------------- -}

	  // The structure of this method has a lot of similarities to
	  // attempt_allocation_slow(). The reason these two were not merged
	  // into a single one is that such a method would require several "if
	  // allocation is not humongous do this, otherwise do that"
	  // conditional paths which would obscure its flow. In fact, an early
	  // version of this code did use a unified method which was harder to
	  // follow and, as a result, it had subtle bugs that were hard to
	  // track down. So keeping these two methods separate allows each to
	  // be more readable. It will be good to keep these two in sync as
	  // much as possible.
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_heap_not_locked_and_not_at_safepoint();
	  assert(isHumongous(word_size), "attempt_allocation_humongous() "
	         "should only be called for humongous allocations");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // We will loop until a) we manage to successfully perform the
	  // allocation or b) we successfully schedule a collection which
	  // fails to perform the allocation. b) is the only case when we'll
	  // return NULL.
	  HeapWord* result = NULL;
	  for (int try_count = 1; /* we'll return */; try_count += 1) {
	    bool should_try_gc;
	    unsigned int gc_count_before;
	
	    {
	      MutexLockerEx x(Heap_lock);
	
	      // Given that humongous objects are not allocated in young
	      // regions, we'll first try to do the allocation without doing a
	      // collection hoping that there's enough space in the heap.
	      result = humongous_obj_allocate(word_size);
	      if (result != NULL) {
	        return result;
	      }
	
	      if (GC_locker::is_active_and_needs_gc()) {
	        should_try_gc = false;
	      } else {
	        // Read the GC count while still holding the Heap_lock.
	        gc_count_before = SharedHeap::heap()->total_collections();
	        should_try_gc = true;
	      }
	    }
	
	    if (should_try_gc) {
	      // If we failed to allocate the humongous object, we should try to
	      // do a collection pause (if we're allowed) in case it reclaims
	      // enough space for the allocation to succeed after the pause.
	
	      bool succeeded;
	      result = do_collection_pause(word_size, gc_count_before, &succeeded);
	      if (result != NULL) {
	        assert(succeeded, "only way to get back a non-NULL result");
	        return result;
	      }
	
	      if (succeeded) {
	        // If we get here we successfully scheduled a collection which
	        // failed to allocate. No point in trying to allocate
	        // further. We'll just return NULL.
	        MutexLockerEx x(Heap_lock);
	        *gc_count_before_ret = SharedHeap::heap()->total_collections();
	        return NULL;
	      }
	    } else {
	      GC_locker::stall_until_clear();
	    }
	
	    // We can reach here if we were unsuccessul in scheduling a
	    // collection (because another thread beat us to it) or if we were
	    // stalled due to the GC locker. In either can we should retry the
	    // allocation attempt in case another thread successfully
	    // performed a collection and reclaimed enough space.  Give a
	    // warning if we seem to be looping forever.
	
	    if ((QueuedAllocationWarningCount > 0) &&
	        (try_count % QueuedAllocationWarningCount == 0)) {
	      warning("G1CollectedHeap::attempt_allocation_humongous() "
	              "retries %d times", try_count);
	    }
	  }
	
	  ShouldNotReachHere();
	  return NULL;
	}
	
```


