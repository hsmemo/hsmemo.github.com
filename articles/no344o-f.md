---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.inline.hpp

### 名前(function name)
```
void CollectedHeap::post_allocation_setup_common(KlassHandle klass,
                                                 HeapWord* obj,
                                                 size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) mark フィールド, 及び klass フィールドを初期化する.
      ---------------------------------------- -}

	  post_allocation_setup_no_klass_install(klass, obj, size);
	  post_allocation_install_obj_klass(klass, oop(obj), (int) size);
	}
	
```


