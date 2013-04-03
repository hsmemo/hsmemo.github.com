---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/aprofiler.cpp

### 名前(function name)
```
void AllocationProfiler::iterate_since_last_gc() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenCollectedHeap::object_iterate_since_last_GC() を呼び出すだけ
      (使用する Closure は AllocProfClosure).
      ---------------------------------------- -}

	  if (is_active()) {
	    AllocProfClosure blk;
	    GenCollectedHeap* heap = GenCollectedHeap::heap();
	    heap->object_iterate_since_last_GC(&blk);
	  }
	}
	
```


