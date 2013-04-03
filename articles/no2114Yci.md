---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.cpp

### 名前(function name)
```
ParCompactionManager*
ParCompactionManager::gc_thread_compaction_manager(int index) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _manager_array から, 引数の index に対応する ParCompactionManager オブジェクトを取り出して, リターンするだけ.
      ---------------------------------------- -}

	  assert(index >= 0 && index < (int)ParallelGCThreads, "index out of range");
	  assert(_manager_array != NULL, "Sanity");
	  return _manager_array[index];
	}
	
```


