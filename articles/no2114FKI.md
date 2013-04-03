---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryService.cpp
### 説明(description)

```
// Add memory pools for ParallelScavengeHeap
// This function currently only supports two generations collected heap.
// The collector for ParallelScavengeHeap will have two memory managers.
```

### 名前(function name)
```
void MemoryService::add_parallel_scavenge_heap_info(ParallelScavengeHeap* heap) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MemoryManager::get_psScavenge_memory_manager() と MemoryManager::get_psMarkSweep_memory_manager() で, 
      それぞれ Minor GC, Major GC 用の MemoryManager を取得する.
      ---------------------------------------- -}

	  // Two managers to keep statistics about _minor_gc_manager and _major_gc_manager GC.
	  _minor_gc_manager = MemoryManager::get_psScavenge_memory_manager();
	  _major_gc_manager = MemoryManager::get_psMarkSweep_memory_manager();
	  _managers_list->append(_minor_gc_manager);
	  _managers_list->append(_major_gc_manager);
	
  {- -------------------------------------------
  (1) MemoryService::add_psYoung_memory_pool(), MemoryService::add_psOld_memory_pool(), MemoryService::add_psPerm_memory_pool() を呼んで, 
      MemoryPool オブジェクトを生成する.
      ---------------------------------------- -}

	  add_psYoung_memory_pool(heap->young_gen(), _major_gc_manager, _minor_gc_manager);
	  add_psOld_memory_pool(heap->old_gen(), _major_gc_manager);
	  add_psPerm_memory_pool(heap->perm_gen(), _major_gc_manager);
	}
	
```


