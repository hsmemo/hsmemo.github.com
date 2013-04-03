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
void MemoryService::add_psOld_memory_pool(PSOldGen* gen, MemoryManager* mgr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PSGenerationPool のインスタンスを生成し, 
      引数で与えられた MemoryManager, 及び _pools_list に登録する.
      ---------------------------------------- -}

	  PSGenerationPool* old_gen = new PSGenerationPool(gen,
	                                                   "PS Old Gen",
	                                                   MemoryPool::Heap,
	                                                   true /* support_usage_threshold */);
	  mgr->add_pool(old_gen);
	  _pools_list->append(old_gen);
	}
	
```


