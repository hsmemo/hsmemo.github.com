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
  void add_reference(OopOrNarrowOopStar from, int tid) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Remembered Set に参照元情報を記録する. 2通りのパスがある.
  
      * もし GC 中であり (= PerRegionTable がセットされており)
        かつこのスレッドが 0 番スレッドでなければ(= tid が 0 でなければ),
        tid に該当する PerRegionTable オブジェクトに対して
        PerRegionTable::seq_add_reference() をよぶことで記録する.
  
      * それ以外の場合は,
        PerRegionTable::add_reference() を呼んで
        base table (= この PosParPRT オブジェクト自身) に記録する.
      ---------------------------------------- -}

	    // Expand if necessary.
	    PerRegionTable** pt = par_tables();
	    if (pt != NULL) {
	      // We always have to assume that mods to table 0 are in parallel,
	      // because of the claiming scheme in parallel expansion.  A thread
	      // with tid != 0 that finds the table to be NULL, but doesn't succeed
	      // in claiming the right of expanding it, will end up in the else
	      // clause of the above if test.  That thread could be delayed, and a
	      // thread 0 add reference could see the table expanded, and come
	      // here.  Both threads would be adding in parallel.  But we get to
	      // not use atomics for tids > 0.
	      if (tid == 0) {
	        PerRegionTable::add_reference(from);
	      } else {
	        pt[tid-1]->seq_add_reference(from);
	      }
	    } else {
	      // Not expanded -- add to the base table.
	      PerRegionTable::add_reference(from);
	    }
	  }
	
```


