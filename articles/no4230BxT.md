---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/constantPoolKlass.cpp

### 名前(function name)
```
constantPoolOop constantPoolKlass::allocate(int length, bool is_conc_safe, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedHeap::permanent_obj_allocate() により Perm 内に領域を確保する (#TODO).
      ---------------------------------------- -}

	  int size = constantPoolOopDesc::object_size(length);
	  KlassHandle klass (THREAD, as_klassOop());
	  assert(klass()->is_oop(), "Can't be null, else handlizing of c below won't work");
	  constantPoolHandle pool;
	  {
	    constantPoolOop c =
	      (constantPoolOop)CollectedHeap::permanent_obj_allocate(klass, size, CHECK_NULL);
	    assert(c->klass_or_null() != NULL, "Handlizing below won't work");
	    pool = constantPoolHandle(THREAD, c);
	  }
	
  {- -------------------------------------------
  (1) その後, 作成した constantPoolOop の各フィールドを初期化.
      ---------------------------------------- -}

	  pool->set_length(length);
	  pool->set_tags(NULL);
	  pool->set_cache(NULL);
	  pool->set_operands(NULL);
	  pool->set_pool_holder(NULL);
	  pool->set_flags(0);
	  // only set to non-zero if constant pool is merged by RedefineClasses
	  pool->set_orig_length(0);
	  // if constant pool may change during RedefineClasses, it is created
	  // unsafe for GC concurrent processing.
	  pool->set_is_conc_safe(is_conc_safe);
	  // all fields are initialized; needed for GC
	
	  // Note: because we may be in this "conc_unsafe" state when allocating
	  // t_oop below, which may in turn cause a GC, it is imperative that our
	  // size be correct, consistent and henceforth stable, at this stage.
	  assert(pool->is_oop() && pool->is_parsable(), "Else size() below is unreliable");
	  assert(size == pool->size(), "size() is wrong");
	
	  // initialize tag array
	  typeArrayOop t_oop = oopFactory::new_permanent_byteArray(length, CHECK_NULL);
	  typeArrayHandle tags (THREAD, t_oop);
	  for (int index = 0; index < length; index++) {
	    tags()->byte_at_put(index, JVM_CONSTANT_Invalid);
	  }
	  pool->set_tags(tags());
	
	  // Check that our size was stable at its old value.
	  assert(size == pool->size(), "size() changed");
	  return pool();
	
```


