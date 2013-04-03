---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryPool.hpp

### 名前(function name)
```
  CollectedMemoryPool(const char* name, PoolType type, size_t init_size, size_t max_size, bool support_usage_threshold) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化
      ---------------------------------------- -}

	    MemoryPool(name, type, init_size, max_size, support_usage_threshold, true) {};
	
```


