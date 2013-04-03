---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp

### 名前(function name)
```
void GCTaskManager::release_all_resources() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 全ての Worker Thread に対して, GCTaskManager::set_resource_flag() を呼んで
      対応する _resource_flag の値を true にする.
      
      (これにより, GCTaskManager::should_release_resources() が true を返すようになる.
       See: GCTaskManager::should_release_resources())
      ---------------------------------------- -}

	  // If you want this to be done atomically, do it in a BarrierGCTask.
	  for (uint i = 0; i < workers(); i += 1) {
	    set_resource_flag(i, true);
	  }
	}
	
```


