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
HeapWord*
G1CollectedHeap::satisfy_failed_allocation(size_t word_size,
                                           bool* succeeded) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_at_safepoint(true /* should_be_vm_thread */);
	
  {- -------------------------------------------
  (1) succeeded 引数が指す箇所を true で初期化しておく.
      ---------------------------------------- -}

	  *succeeded = true;

  {- -------------------------------------------
  (1) まず G1CollectedHeap::attempt_allocation_at_safepoint() で確保を試みる.
      確保に成功したら, ここでリターン.
      ---------------------------------------- -}

	  // Let's attempt the allocation first.
	  HeapWord* result =
	    attempt_allocation_at_safepoint(word_size,
	                                 false /* expect_null_mutator_alloc_region */);
	  if (result != NULL) {
	    assert(*succeeded, "sanity");
	    return result;
	  }
	
  {- -------------------------------------------
  (1) まず G1CollectedHeap::expand_and_allocate() で確保を試みる.
      確保に成功したら, ここでリターン.
      ---------------------------------------- -}

	  // In a G1 heap, we're supposed to keep allocation from failing by
	  // incremental pauses.  Therefore, at least for now, we'll favor
	  // expansion over collection.  (This might change in the future if we can
	  // do something smarter than full collection to satisfy a failed alloc.)
	  result = expand_and_allocate(word_size);
	  if (result != NULL) {
	    assert(*succeeded, "sanity");
	    return result;
	  }
	
  {- -------------------------------------------
  (1) G1CollectedHeap::do_collection() を呼んで, Full GC を実行する.
      Full GC が失敗したら, succeeded 引数が指す箇所を false に変更した後, ここで NULL をリターン.
      ---------------------------------------- -}

	  // Expansion didn't work, we'll try to do a Full GC.
	  bool gc_succeeded = do_collection(false, /* explicit_gc */
	                                    false, /* clear_all_soft_refs */
	                                    word_size);
	  if (!gc_succeeded) {
	    *succeeded = false;
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) もう一度 G1CollectedHeap::attempt_allocation_at_safepoint() で確保を試みる.
      確保に成功したら, ここでリターン.
      ---------------------------------------- -}

	  // Retry the allocation
	  result = attempt_allocation_at_safepoint(word_size,
	                                  true /* expect_null_mutator_alloc_region */);
	  if (result != NULL) {
	    assert(*succeeded, "sanity");
	    return result;
	  }
	
  {- -------------------------------------------
  (1) G1CollectedHeap::do_collection() を呼んで, もう一度 Full GC を実行する.
      (今度は soft reference は全部消すことにする)
      Full GC が失敗したら, succeeded 引数が指す箇所を false に変更した後, ここで NULL をリターン.
      ---------------------------------------- -}

	  // Then, try a Full GC that will collect all soft references.
	  gc_succeeded = do_collection(false, /* explicit_gc */
	                               true,  /* clear_all_soft_refs */
	                               word_size);
	  if (!gc_succeeded) {
	    *succeeded = false;
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) もう一度 G1CollectedHeap::attempt_allocation_at_safepoint() で確保を試みる.
      確保に成功したら, ここでリターン.
      ---------------------------------------- -}

	  // Retry the allocation once more
	  result = attempt_allocation_at_safepoint(word_size,
	                                  true /* expect_null_mutator_alloc_region */);
	  if (result != NULL) {
	    assert(*succeeded, "sanity");
	    return result;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!collector_policy()->should_clear_all_soft_refs(),
	         "Flag should have been handled and cleared prior to this point");
	
	  // What else?  We might try synchronous finalization later.  If the total
	  // space available is large enough for the allocation, then a more
	  // complete compaction phase than we've tried so far might be
	  // appropriate.
	  assert(*succeeded, "sanity");

  {- -------------------------------------------
  (1) NULL をリターン.
      ---------------------------------------- -}

	  return NULL;
	}
	
```


