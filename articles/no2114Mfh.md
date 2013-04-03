---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.hpp
### 説明(description)

```
  // low memory detection for collected memory pools.
```

### 名前(function name)
```
  static inline void detect_low_memory_for_collected_pools() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし閾値超過検出機能が有効になっていなければ (= LowMemoryDetector::is_enabled_for_collected_pools() が false ならば) 
      何もすることはないので, ここでリターン.
      ---------------------------------------- -}

	    // no-op if low memory detection not enabled
	    if (!is_enabled_for_collected_pools()) {
	      return;
	    }

  {- -------------------------------------------
  (1) CollectedMemoryPool のサブクラスである全ての MemoryPool に対して,  
      LowMemoryDetector::is_enabled() を呼んで
      閾値超過通知機能が有効になっているかどうかを調べていく.
    
      有効になっているものについては, 現在の使用量と usage threshold の high threshold を比較し, 
      使用量の方が上回っていれば, LowMemoryDetector::detect_low_memory() でさらなる確認&通知処理を行う.
      ---------------------------------------- -}

	    int num_memory_pools = MemoryService::num_memory_pools();
	    for (int i=0; i<num_memory_pools; i++) {
	      MemoryPool* pool = MemoryService::get_memory_pool(i);
	
	      // if low memory detection is enabled then check if the
	      // current used exceeds the high threshold
	      if (pool->is_collected_pool() && is_enabled(pool)) {
	        size_t used = pool->used_in_bytes();
	        size_t high = pool->usage_threshold()->high_threshold();
	        if (used > high) {
	          detect_low_memory(pool);
	        }
	      }
	    }
	  }
	
```


