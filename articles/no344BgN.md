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
oop CollectedHeap::obj_allocate(KlassHandle klass, int size, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  debug_only(check_for_valid_allocation_state());
	  assert(!Universe::heap()->is_gc_active(), "Allocation during gc not allowed");
	  assert(size >= 0, "int won't convert to size_t");

  {- -------------------------------------------
  (1) CollectedHeap::common_mem_allocate_init() でメモリを確保する
      ---------------------------------------- -}

	  HeapWord* obj = common_mem_allocate_init(size, false, CHECK_NULL);

  {- -------------------------------------------
  (1) CollectedHeap::post_allocation_setup_array() で確保したメモリのヘッダー部分を初期化する 
      (mark フィールド, klass フィールド)
      (また, JVMTI や DTrace, JMM のフック点でもある)
      ---------------------------------------- -}

	  post_allocation_setup_obj(klass, obj, size);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  NOT_PRODUCT(Universe::heap()->check_for_bad_heap_word_value(obj, size));

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return (oop)obj;
	}
	
```


