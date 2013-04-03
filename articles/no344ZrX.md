---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.inline.hpp

### 名前(function name)
```
inline HeapWord*
G1CollectedHeap::attempt_allocation(size_t word_size,
                                    unsigned int* gc_count_before_ret) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_heap_not_locked_and_not_at_safepoint();
	  assert(!isHumongous(word_size), "attempt_allocation() should not "
	         "be called for humongous allocation requests");
	
  {- -------------------------------------------
  (1) G1AllocRegion::attempt_allocation() でメモリ確保を試みる.
      ---------------------------------------- -}

	  HeapWord* result = _mutator_alloc_region.attempt_allocation(word_size,
	                                                      false /* bot_updates */);

  {- -------------------------------------------
  (1) もし確保に失敗していれば, G1CollectedHeap::attempt_allocation_slow() でメモリ確保を試みる.
      ---------------------------------------- -}

	  if (result == NULL) {
	    result = attempt_allocation_slow(word_size, gc_count_before_ret);
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_heap_not_locked();

  {- -------------------------------------------
  (1) 以上の処理で確保に成功していたら, barrier set を dirty 化しておく
      ---------------------------------------- -}

	  if (result != NULL) {
	    dirty_young_block(result, word_size);
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return result;
	}
	
```


