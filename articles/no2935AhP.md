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
HeapWord* G1CollectedHeap::attempt_allocation_at_safepoint(size_t word_size,
                                       bool expect_null_mutator_alloc_region) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_at_safepoint(true /* should_be_vm_thread */);
	  assert(_mutator_alloc_region.get() == NULL ||
	                                             !expect_null_mutator_alloc_region,
	         "the current alloc region was unexpectedly found to be non-NULL");
	
  {- -------------------------------------------
  (1) もし確保サイズが大き過ぎなければ(= isHumongous() が false ならば), 
      G1AllocRegion::attempt_allocation_locked() でメモリ確保を試み, 結果をリターンする.
    
      逆に大き過ぎる場合は, 
      G1CollectedHeap::humongous_obj_allocate() を呼んでメモリ確保を試み, 結果をリターンする.
      ---------------------------------------- -}

	  if (!isHumongous(word_size)) {
	    return _mutator_alloc_region.attempt_allocation_locked(word_size,
	                                                      false /* bot_updates */);
	  } else {
	    return humongous_obj_allocate(word_size);
	  }
	
  {- -------------------------------------------
  (1) (以下のパスに来ることはあり得ない)
      ---------------------------------------- -}

	  ShouldNotReachHere();
	}
	
```


