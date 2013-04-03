---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/g1MemoryPool.cpp

### 名前(function name)
```
G1MemoryPoolSuper::G1MemoryPoolSuper(G1CollectedHeap* g1h,
                                     const char* name,
                                     size_t init_size,
                                     bool support_usage_threshold) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化, (フィールドの初期化)
      ---------------------------------------- -}

	  _g1h(g1h), CollectedMemoryPool(name,
	                                   MemoryPool::Heap,
	                                   init_size,
	                                   undefined_max(),
	                                   support_usage_threshold) {

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(UseG1GC, "sanity");
	}
	
```


