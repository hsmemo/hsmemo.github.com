---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp

### 名前(function name)
```
  void add_reference(OopOrNarrowOopStar from) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PerRegionTable::add_reference_work() を呼び出すだけ.
      ---------------------------------------- -}

	    add_reference_work(from, /*parallel*/ true);
	  }
	
```


