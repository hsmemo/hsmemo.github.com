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
GCMemoryManager* MemoryManager::get_copy_memory_manager() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい CopyMemoryManager のインスタンスを作ってリターンするだけ.
      ---------------------------------------- -}

	  return (GCMemoryManager*) new CopyMemoryManager();
	}
	
```


