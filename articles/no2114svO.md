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
void MemoryService::add_g1OldGen_memory_pool(G1CollectedHeap* g1h,
                                             MemoryManager* mgr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) G1OldGenPool のインスタンスを生成し, 
      引数で与えられた MemoryManager, 及び _pools_list に登録する.
      ---------------------------------------- -}

	  assert(mgr != NULL, "should have one manager");
	
	  G1OldGenPool* old_gen = new G1OldGenPool(g1h);
	  mgr->add_pool(old_gen);
	  _pools_list->append(old_gen);
	}
	
```


