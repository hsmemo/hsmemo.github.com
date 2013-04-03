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
void CollectedHeap::post_allocation_setup_array(KlassHandle klass,
                                                HeapWord* obj,
                                                size_t size,
                                                int length) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, Concurrent GC とうまくやるためには length を先に設定する必要があるとのこと)
      ---------------------------------------- -}

	  // Set array length before setting the _klass field
	  // in post_allocation_setup_common() because the klass field
	  // indicates that the object is parsable by concurrent GC.

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(length >= 0, "length should be non-negative");

  {- -------------------------------------------
  (1) length フィールドを初期化する.
      ---------------------------------------- -}

	  ((arrayOop)obj)->set_length(length);

  {- -------------------------------------------
  (1) CollectedHeap::post_allocation_setup_common() で 
      mark フィールドと klass フィールドを初期化する.
      ---------------------------------------- -}

	  post_allocation_setup_common(klass, obj, size);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(((oop)obj)->blueprint()->oop_is_array(), "must be an array");

  {- -------------------------------------------
  (1) (JVMTI や DTrace, JMM のフック点)
      (See: post_allocation_notify())
      ---------------------------------------- -}

	  // notify jvmti and dtrace (must be after length is set for dtrace)
	  post_allocation_notify(klass, (oop)obj);
	}
	
```


