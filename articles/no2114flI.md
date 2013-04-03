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
void MemoryService::add_g1YoungGen_memory_pool(G1CollectedHeap* g1h,
                                               MemoryManager* major_mgr,
                                               MemoryManager* minor_mgr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) G1EdenPool 及び G1SurvivorPool のインスタンスを生成し, 
      引数で与えられた MemoryManager, 及び _pools_list に登録する.
      ---------------------------------------- -}

	  assert(major_mgr != NULL && minor_mgr != NULL, "should have two managers");
	
	  G1EdenPool* eden = new G1EdenPool(g1h);
	  G1SurvivorPool* survivor = new G1SurvivorPool(g1h);
	
	  major_mgr->add_pool(eden);
	  major_mgr->add_pool(survivor);
	  minor_mgr->add_pool(eden);
	  minor_mgr->add_pool(survivor);
	  _pools_list->append(eden);
	  _pools_list->append(survivor);
	}
	
```


