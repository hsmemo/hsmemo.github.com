---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryPool.cpp

### 名前(function name)
```
MemoryPool::MemoryPool(const char* name,
                       PoolType type,
                       size_t init_size,
                       size_t max_size,
                       bool support_usage_threshold,
                       bool support_gc_threshold) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (フィールドの初期化)
      ---------------------------------------- -}

	  _name = name;
	  _initial_size = init_size;
	  _max_size = max_size;
	  _memory_pool_obj = NULL;
	  _available_for_allocation = true;
	  _num_managers = 0;
	  _type = type;
	
	  // initialize the max and init size of collection usage
	  _after_gc_usage = MemoryUsage(_initial_size, 0, 0, _max_size);
	
	  _usage_sensor = NULL;
	  _gc_usage_sensor = NULL;
	  // usage threshold supports both high and low threshold
	  _usage_threshold = new ThresholdSupport(support_usage_threshold, support_usage_threshold);
	  // gc usage threshold supports only high threshold
	  _gc_usage_threshold = new ThresholdSupport(support_gc_threshold, support_gc_threshold);
	}
	
```


