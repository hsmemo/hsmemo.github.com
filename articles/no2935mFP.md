---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.cpp

### 名前(function name)
```
void HeapRegionSeq::iterate(HeapRegionClosure* blk) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) HeapRegionSeq::iterate_from(HeapRegion* r, HeapRegionClosure* blk) を呼び出すだけ.
      ---------------------------------------- -}

	  iterate_from((HeapRegion*)NULL, blk);
	}
	
```


