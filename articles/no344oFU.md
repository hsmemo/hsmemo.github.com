---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.inline.hpp

### 名前(function name)
```
HeapWord* CollectedHeap::common_mem_allocate_init(size_t size, bool is_noref, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedHeap::common_mem_allocate_noinit() でメモリを確保する.
      ---------------------------------------- -}

	  HeapWord* obj = common_mem_allocate_noinit(size, is_noref, CHECK_NULL);

  {- -------------------------------------------
  (1) CollectedHeap::init_obj() で, 確保したメモリのヘッダー部以外の領域を 0 クリアする.
      ---------------------------------------- -}

	  init_obj(obj, size);

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return obj;
	}
	
```


