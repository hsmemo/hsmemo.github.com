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
  // indicates if low memory detection is enabled for any collected
  // memory pools
```

### 名前(function name)
```
  static inline bool is_enabled_for_collected_pools() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) LowMemoryDetectorDisabler オブジェクトによって一時的に disabled されておらず, 
      かつ一つ以上の CollectedMemoryPool で閾値超過機能が有効になっていれば 
      (= _enabled_for_collected_pools フィールドが true であれば),  
      true を返す.
      ---------------------------------------- -}

	    return !temporary_disabled() && _enabled_for_collected_pools;
	  }
	
```


