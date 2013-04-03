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
    void set_partial_obj_size(size_t words)    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _partial_obj_size フィールドをセットするだけ.
      ---------------------------------------- -}

	      _partial_obj_size = (region_sz_t) words;
	    }
	
```


