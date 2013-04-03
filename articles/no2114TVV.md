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
MemoryPool* MemoryService::add_cms_space(CompactibleFreeListSpace* space,
                                         const char* name,
                                         bool is_heap,
                                         size_t max_size,
                                         bool support_usage_threshold) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CompactibleFreeListSpacePool のインスタンスを生成し, _pools_list に登録する.
      ---------------------------------------- -}

	  MemoryPool::PoolType type = (is_heap ? MemoryPool::Heap : MemoryPool::NonHeap);
	  CompactibleFreeListSpacePool* pool = new CompactibleFreeListSpacePool(space, name, type, max_size, support_usage_threshold);
	  _pools_list->append(pool);
	  return (MemoryPool*) pool;
	}
	
```


