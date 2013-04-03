---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.cpp
### 説明(description)

```
// recompute enabled flag
```

### 名前(function name)
```
void LowMemoryDetector::recompute_enabled_for_collected_pools() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedMemoryPool のサブクラスである全ての MemoryPool に対して,  
      LowMemoryDetector::is_enabled() を呼んで
      閾値超過通知機能が有効になっているかどうかを調べていく.
    
      もし1つでも有効になっているものがあれば, 
      _enabled_for_collected_pools フィールドを true に設定する.
      逆に, 有効になっているものが1つもなければ false に設定する.
      ---------------------------------------- -}

	  bool enabled = false;
	  int num_memory_pools = MemoryService::num_memory_pools();
	  for (int i=0; i<num_memory_pools; i++) {
	    MemoryPool* pool = MemoryService::get_memory_pool(i);
	    if (pool->is_collected_pool() && is_enabled(pool)) {
	      enabled = true;
	      break;
	    }
	  }
	  _enabled_for_collected_pools = enabled;
	}
	
```


