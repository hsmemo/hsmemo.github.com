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
void CollectedHeap::post_allocation_setup_obj(KlassHandle klass,
                                              HeapWord* obj,
                                              size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedHeap::post_allocation_setup_common() で 
      mark フィールドと klass フィールドを初期化する.
      ---------------------------------------- -}

	  post_allocation_setup_common(klass, obj, size);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Universe::is_bootstrapping() ||
	         !((oop)obj)->blueprint()->oop_is_array(), "must not be an array");

  {- -------------------------------------------
  (1) (JVMTI や DTrace, JMM のフック点)
      (See: post_allocation_notify())
      ---------------------------------------- -}

	  // notify jvmti and dtrace
	  post_allocation_notify(klass, (oop)obj);
	}
	
```


