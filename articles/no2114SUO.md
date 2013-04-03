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
void MemoryService::add_g1_heap_info(G1CollectedHeap* g1h) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(UseG1GC, "sanity");
	
  {- -------------------------------------------
  (1) MemoryManager::get_g1YoungGen_memory_manager() と MemoryManager::get_g1OldGen_memory_manager() で, 
      それぞれ Minor GC, Major GC 用の MemoryManager を取得する.
      ---------------------------------------- -}

	  _minor_gc_manager = MemoryManager::get_g1YoungGen_memory_manager();
	  _major_gc_manager = MemoryManager::get_g1OldGen_memory_manager();
	  _managers_list->append(_minor_gc_manager);
	  _managers_list->append(_major_gc_manager);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  add_g1YoungGen_memory_pool(g1h, _major_gc_manager, _minor_gc_manager);
	  add_g1OldGen_memory_pool(g1h, _major_gc_manager);
	  add_g1PermGen_memory_pool(g1h, _major_gc_manager);
	}
	
```


