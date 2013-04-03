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
void MemoryService::gc_begin(bool fullGC, bool recordGCBeginTime,
                             bool recordAccumulatedGCTime,
                             bool recordPreGCUsage, bool recordPeakUsage) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GCMemoryManager::gc_begin() を呼び出す.
      どちらのオブジェクトに対して呼び出すかは, 引数の fullGC の値に応じて決める.
      (full GC の場合には _major_gc_manager, そうでなければ _minor_gc_manager)
      ---------------------------------------- -}
	
	  GCMemoryManager* mgr;
	  if (fullGC) {
	    mgr = _major_gc_manager;
	  } else {
	    mgr = _minor_gc_manager;
	  }
	  assert(mgr->is_gc_memory_manager(), "Sanity check");
	  mgr->gc_begin(recordGCBeginTime, recordPreGCUsage, recordAccumulatedGCTime);
	
  {- -------------------------------------------
  (1) 各 MemoryPool に対して MemoryPool::record_peak_memory_usage() を呼び出し, 最大使用量等の情報を更新する.
  
      (ただし, 引数の recordPeakUsage が false であれば, この処理は行わない)
      ---------------------------------------- -}

	  // Track the peak memory usage when GC begins
	  if (recordPeakUsage) {
	    for (int i = 0; i < _pools_list->length(); i++) {
	      MemoryPool* pool = _pools_list->at(i);
	      pool->record_peak_memory_usage();
	    }
	  }
	}
	
```


