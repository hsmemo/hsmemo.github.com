---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp

### 名前(function name)
```
  void fill(ParallelScavengeHeap* heap) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの値を各ヒープ領域の使用量で初期化する.
      ---------------------------------------- -}

	    _heap_used      = heap->used();
	    _young_gen_used = heap->young_gen()->used_in_bytes();
	    _old_gen_used   = heap->old_gen()->used_in_bytes();
	    _perm_gen_used  = heap->perm_gen()->used_in_bytes();
	  };
	
```


