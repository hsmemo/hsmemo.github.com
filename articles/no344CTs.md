---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.inline.hpp


### 本体部(body)
```
	void CollectedHeap::post_allocation_install_obj_klass(KlassHandle klass,
	                                                   oop obj,
	                                                   int size) {

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // These asserts are kind of complicated because of klassKlass
	  // and the beginning of the world.
	  assert(klass() != NULL || !Universe::is_fully_initialized(), "NULL klass");
	  assert(klass() == NULL || klass()->is_klass(), "not a klass");
	  assert(klass() == NULL || klass()->klass_part() != NULL, "not a klass");
	  assert(obj != NULL, "NULL object pointer");

  {- -------------------------------------------
  (1) klass フィールドを初期化する
      ---------------------------------------- -}

	  obj->set_klass(klass());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!Universe::is_fully_initialized() || obj->blueprint() != NULL,
	         "missing blueprint");
	}
	
```


