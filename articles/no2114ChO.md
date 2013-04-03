---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp

### 名前(function name)
```
  virtual IterationStatus do_addr(HeapWord* addr, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定された範囲に CollectedHeap::fill_with_objects() でダミーのオブジェクトを詰める.
      (ついでに, 対応する範囲の offset array (object start array) の情報も更新する)
      ---------------------------------------- -}

	    CollectedHeap::fill_with_objects(addr, size);
	    HeapWord* const end = addr + size;
	    do {
	      _start_array->allocate_block(addr);
	      addr += oop(addr)->size();
	    } while (addr < end);

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	    return ParMarkBitMap::incomplete;
	  }
	
```


