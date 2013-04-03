---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryManager.cpp

### 名前(function name)
```
MemoryManager* MemoryManager::get_code_cache_memory_manager() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい CodeCacheMemoryManager のインスタンスを作ってリターンするだけ.
      ---------------------------------------- -}

	  return (MemoryManager*) new CodeCacheMemoryManager();
	}
	
```


