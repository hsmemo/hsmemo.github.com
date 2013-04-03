---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.inline.hpp


### 本体部(body)
```
	inline HeapWord* G1AllocRegion::par_allocate(HeapRegion* alloc_region,
	                                             size_t word_size,
	                                             bool bot_updates) {

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(alloc_region != NULL, err_msg("pre-condition"));
	  assert(!alloc_region->is_empty(), err_msg("pre-condition"));
	
  {- -------------------------------------------
  (1) 引数の bot_update の値が false ならば HeapRegion::par_allocate_no_bot_updates(), 
      true ならば G1OffsetTableContigSpace::par_allocate() で確保処理を行い, 
      結果をリターン.
      ---------------------------------------- -}

	  if (!bot_updates) {
	    return alloc_region->par_allocate_no_bot_updates(word_size);
	  } else {
	    return alloc_region->par_allocate(word_size);
	  }
	}
	
```


