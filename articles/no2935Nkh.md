---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp

### 名前(function name)
```
  bool doHeapRegion(HeapRegion* r) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数を _a_region フィールドに格納して true をリターンするだけ.
  
      (true をリターンすることで G1CollectedHeap::heap_region_iterate() が停止するため, 
       結果的に一番最初の HeapRegion がフィールドに記録される)
      ---------------------------------------- -}

	    _a_region = r;
	    return true;
	  }
	
```


