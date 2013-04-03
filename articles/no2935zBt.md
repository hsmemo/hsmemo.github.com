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
void G1CollectedHeap::collect(GCCause::Cause cause) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // The caller doesn't have the Heap_lock
	  assert(!Heap_lock->owned_by_self(), "this thread should not own the Heap_lock");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  unsigned int gc_count_before;
	  unsigned int full_gc_count_before;

  {- -------------------------------------------
  (1) total_collections() と total_full_collections() の回数を取得する.
      (なお, この処理は Heap_lock で排他して行う)
      ---------------------------------------- -}

	  {
	    MutexLocker ml(Heap_lock);
	
	    // Read the GC count while holding the Heap_lock
	    gc_count_before = SharedHeap::heap()->total_collections();
	    full_gc_count_before = SharedHeap::heap()->total_full_collections();
	  }
	
  {- -------------------------------------------
  (1) cause 引数の値に応じて以下のどれかを行う.
  
      * Concurrent Full GC を実行すべき場合
        (= cause の値に対して G1CollectedHeap::should_do_concurrent_full_gc() が true を返す場合): 
  
        VM_G1IncCollectionPause で initial-mark evacuation pause を実行し, 
        それによって Concurrent Mark 処理を開始させる.
  
      * Concurrent Full GC の必要はなく, 呼び出し元が GC_locker (= cause 引数が GCCause::_gc_locker) の場合:
  
        VM_G1IncCollectionPause で Minor GC を実行.
        
      * Concurrent Full GC の必要はなく, 呼び出し元が GC_locker 以外の場合: 
  
        VM_G1CollectFull で Major GC を実行.
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (以下が "Concurrent Full GC を実行すべき場合" の処理)
        ---------------------------------------- -}

	  if (should_do_concurrent_full_gc(cause)) {
	    // Schedule an initial-mark evacuation pause that will start a
	    // concurrent cycle. We're setting word_size to 0 which means that
	    // we are not requesting a post-GC allocation.
	    VM_G1IncCollectionPause op(gc_count_before,
	                               0,     /* word_size */
	                               true,  /* should_initiate_conc_mark */
	                               g1_policy()->max_pause_time_ms(),
	                               cause);
	    VMThread::execute(&op);

    {- -------------------------------------------
  (1.1) (以下が "Concurrent Full GC の必要はなく, 呼び出し元が GC_locker の場合" の処理)
        ---------------------------------------- -}

	  } else {
	    if (cause == GCCause::_gc_locker
	        DEBUG_ONLY(|| cause == GCCause::_scavenge_alot)) {
	
	      // Schedule a standard evacuation pause. We're setting word_size
	      // to 0 which means that we are not requesting a post-GC allocation.
	      VM_G1IncCollectionPause op(gc_count_before,
	                                 0,     /* word_size */
	                                 false, /* should_initiate_conc_mark */
	                                 g1_policy()->max_pause_time_ms(),
	                                 cause);
	      VMThread::execute(&op);

    {- -------------------------------------------
  (1.1) (以下が "Concurrent Full GC の必要はなく, 呼び出し元が GC_locker 以外の場合" の処理)
        ---------------------------------------- -}

	    } else {
	      // Schedule a Full GC.
	      VM_G1CollectFull op(gc_count_before, full_gc_count_before, cause);
	      VMThread::execute(&op);
	    }
	  }
	}
	
```


