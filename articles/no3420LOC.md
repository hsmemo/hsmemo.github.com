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
void
OtherRegionsTable::do_cleanup_work(HRRSCleanupTask* hrrs_cleanup_task) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SparsePRT::do_cleanup_work() を呼び出すだけ.
      ---------------------------------------- -}

	  _sparse_table.do_cleanup_work(hrrs_cleanup_task);
	}
	
```


