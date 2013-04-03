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
void MemoryService::add_code_heap_memory_pool(CodeHeap* heap) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CodeHeapPool のインスタンスを生成する.
      その後, そのインスタンスを
      MemoryManager::get_code_cache_memory_manager() で生成した CodeCacheMemoryManager, 及び 
      _pools_list に登録する.
      ---------------------------------------- -}

	  _code_heap_pool = new CodeHeapPool(heap,
	                                     "Code Cache",
	                                     true /* support_usage_threshold */);
	  MemoryManager* mgr = MemoryManager::get_code_cache_memory_manager();
	  mgr->add_pool(_code_heap_pool);
	
	  _pools_list->append(_code_heap_pool);
	  _managers_list->append(mgr);
	}
	
```


