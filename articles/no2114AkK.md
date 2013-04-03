---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryService.cpp

### 名前(function name)
```
TraceMemoryManagerStats::~TraceMemoryManagerStats() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MemoryService::gc_end() を呼び出すだけ.
      ---------------------------------------- -}

	  MemoryService::gc_end(_fullGC, _recordPostGCUsage, _recordAccumulatedGCTime,
	                        _recordGCEndTime, _countCollection, _cause);
	}
	
```


