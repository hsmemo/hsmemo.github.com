---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.cpp

### 名前(function name)
```
FilterOutOfRegionClosure::FilterOutOfRegionClosure(HeapRegion* r,
                                                   OopClosure* oc) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _r_bottom(r->bottom()), _r_end(r->end()),
	  _oc(oc), _out_of_region(0)
	{}
	
```


