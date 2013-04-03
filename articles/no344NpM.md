---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/genCollectedHeap.cpp

### 名前(function name)
```
HeapWord* GenCollectedHeap::mem_allocate(size_t size,
                                         bool is_large_noref,
                                         bool is_tlab,
                                         bool* gc_overhead_limit_was_exceeded) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenCollectorPolicy::mem_allocate_work() を呼び出すだけ.
      ---------------------------------------- -}

	  return collector_policy()->mem_allocate_work(size,
	                                               is_tlab,
	                                               gc_overhead_limit_was_exceeded);
	}
	
```


