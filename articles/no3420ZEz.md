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
void HeapRegionRemSet::do_cleanup_work(HRRSCleanupTask* hrrs_cleanup_task) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) OtherRegionsTable::do_cleanup_work() を呼び出すだけ.
      ---------------------------------------- -}

	  _other_regions.do_cleanup_work(hrrs_cleanup_task);
	}
	
```


