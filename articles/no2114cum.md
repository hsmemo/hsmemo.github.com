---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp

### 名前(function name)
```
  static void fill_and_update_region(ParCompactionManager* cm, size_t region) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PSParallelCompact::fill_region() を呼び出すだけ.
      ---------------------------------------- -}

	    fill_region(cm, region);
	  }
	
```


