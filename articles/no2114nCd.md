---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryManager.cpp

### 名前(function name)
```
void GCMemoryManager::gc_begin(bool recordGCBeginTime, bool recordPreGCUsage,
                               bool recordAccumulatedGCTime) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_last_gc_stat != NULL && _current_gc_stat != NULL, "Just checking");

  {- -------------------------------------------
  (1) (引数の recordAccumulatedGCTime が指定されていれば) GC 開始時の時刻を記録しておく.
      (See: sun.management.GarbageCollectorImpl.getCollectionTime())
      ---------------------------------------- -}

	  if (recordAccumulatedGCTime) {
	    _accumulated_timer.start();
	  }

  {- -------------------------------------------
  (1) (引数の recordGCBeginTime が指定されていれば) GC 開始時の時刻を記録しておく.
      (See: GCStatInfo)
      ---------------------------------------- -}

	  // _num_collections now increases in gc_end, to count completed collections
	  if (recordGCBeginTime) {
	    _current_gc_stat->set_index(_num_collections+1);
	    _current_gc_stat->set_start_time(Management::timestamp());
	  }
	
  {- -------------------------------------------
  (1) (引数の recordPreGCUsage が指定されていれば) 
      全ての MemoryPool に対して, GCStatInfo::set_before_gc_usage() を呼び出して
      GC 開始時の使用量を記録しておく.
      
      ついでに, (DTrace のフック点)でもある.
      ---------------------------------------- -}

	  if (recordPreGCUsage) {
	    // Keep memory usage of all memory pools
	    for (int i = 0; i < MemoryService::num_memory_pools(); i++) {
	      MemoryPool* pool = MemoryService::get_memory_pool(i);
	      MemoryUsage usage = pool->get_memory_usage();
	      _current_gc_stat->set_before_gc_usage(i, usage);
	      HS_DTRACE_PROBE8(hotspot, mem__pool__gc__begin,
	        name(), strlen(name()),
	        pool->name(), strlen(pool->name()),
	        usage.init_size(), usage.used(),
	        usage.committed(), usage.max_size());
	    }
	  }
	}
	
```


