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
void MemoryService::add_psYoung_memory_pool(PSYoungGen* gen, MemoryManager* major_mgr, MemoryManager* minor_mgr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) EdenMutableSpacePool と SurvivorMutableSpacePool のインスタンスを生成し, 
      引数で与えられた MemoryManager, 及び _pools_list に登録する.
      ---------------------------------------- -}

	  assert(major_mgr != NULL && minor_mgr != NULL, "Should have two managers");
	
	  // Add a memory pool for each space and young gen doesn't
	  // support low memory detection as it is expected to get filled up.
	  EdenMutableSpacePool* eden = new EdenMutableSpacePool(gen,
	                                                        gen->eden_space(),
	                                                        "PS Eden Space",
	                                                        MemoryPool::Heap,
	                                                        false /* support_usage_threshold */);
	
	  SurvivorMutableSpacePool* survivor = new SurvivorMutableSpacePool(gen,
	                                                                    "PS Survivor Space",
	                                                                    MemoryPool::Heap,
	                                                                    false /* support_usage_threshold */);
	
	  major_mgr->add_pool(eden);
	  major_mgr->add_pool(survivor);
	  minor_mgr->add_pool(eden);
	  minor_mgr->add_pool(survivor);
	  _pools_list->append(eden);
	  _pools_list->append(survivor);
	}
	
```


