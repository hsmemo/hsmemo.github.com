---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/universe.cpp

### 名前(function name)
```
bool universe_post_init() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!is_init_completed(), "Error: initialization not yet completed!");

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  Universe::_fully_initialized = true;
	  EXCEPTION_MARK;
	  { ResourceMark rm;
	    Interpreter::initialize();      // needed for interpreter entry points
	    if (!UseSharedSpaces) {
	      KlassHandle ok_h(THREAD, SystemDictionary::Object_klass());
	      Universe::reinitialize_vtable_of(ok_h, CHECK_false);
	      Universe::reinitialize_itables(CHECK_false);
	    }
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  klassOop k;
	  instanceKlassHandle k_h;
	  if (!UseSharedSpaces) {
	    // Setup preallocated empty java.lang.Class array
	    Universe::_the_empty_class_klass_array = oopFactory::new_objArray(SystemDictionary::Class_klass(), 0, CHECK_false);
	    // Setup preallocated OutOfMemoryError errors
	    k = SystemDictionary::resolve_or_fail(vmSymbols::java_lang_OutOfMemoryError(), true, CHECK_false);
	    k_h = instanceKlassHandle(THREAD, k);
	    Universe::_out_of_memory_error_java_heap = k_h->allocate_permanent_instance(CHECK_false);
	    Universe::_out_of_memory_error_perm_gen = k_h->allocate_permanent_instance(CHECK_false);
	    Universe::_out_of_memory_error_array_size = k_h->allocate_permanent_instance(CHECK_false);
	    Universe::_out_of_memory_error_gc_overhead_limit =
	      k_h->allocate_permanent_instance(CHECK_false);
	
	    // Setup preallocated NullPointerException
	    // (this is currently used for a cheap & dirty solution in compiler exception handling)
	    k = SystemDictionary::resolve_or_fail(vmSymbols::java_lang_NullPointerException(), true, CHECK_false);
	    Universe::_null_ptr_exception_instance = instanceKlass::cast(k)->allocate_permanent_instance(CHECK_false);
	    // Setup preallocated ArithmeticException
	    // (this is currently used for a cheap & dirty solution in compiler exception handling)
	    k = SystemDictionary::resolve_or_fail(vmSymbols::java_lang_ArithmeticException(), true, CHECK_false);
	    Universe::_arithmetic_exception_instance = instanceKlass::cast(k)->allocate_permanent_instance(CHECK_false);
	    // Virtual Machine Error for when we get into a situation we can't resolve
	    k = SystemDictionary::resolve_or_fail(
	      vmSymbols::java_lang_VirtualMachineError(), true, CHECK_false);
	    bool linked = instanceKlass::cast(k)->link_class_or_fail(CHECK_false);
	    if (!linked) {
	      tty->print_cr("Unable to link/verify VirtualMachineError class");
	      return false; // initialization failed
	    }
	    Universe::_virtual_machine_error_instance =
	      instanceKlass::cast(k)->allocate_permanent_instance(CHECK_false);
	
	    Universe::_vm_exception               = instanceKlass::cast(k)->allocate_permanent_instance(CHECK_false);
	
	  }
	  if (!DumpSharedSpaces) {
	    // These are the only Java fields that are currently set during shared space dumping.
	    // We prefer to not handle this generally, so we always reinitialize these detail messages.
	    Handle msg = java_lang_String::create_from_str("Java heap space", CHECK_false);
	    java_lang_Throwable::set_message(Universe::_out_of_memory_error_java_heap, msg());
	
	    msg = java_lang_String::create_from_str("PermGen space", CHECK_false);
	    java_lang_Throwable::set_message(Universe::_out_of_memory_error_perm_gen, msg());
	
	    msg = java_lang_String::create_from_str("Requested array size exceeds VM limit", CHECK_false);
	    java_lang_Throwable::set_message(Universe::_out_of_memory_error_array_size, msg());
	
	    msg = java_lang_String::create_from_str("GC overhead limit exceeded", CHECK_false);
	    java_lang_Throwable::set_message(Universe::_out_of_memory_error_gc_overhead_limit, msg());
	
	    msg = java_lang_String::create_from_str("/ by zero", CHECK_false);
	    java_lang_Throwable::set_message(Universe::_arithmetic_exception_instance, msg());
	
	    // Setup the array of errors that have preallocated backtrace
	    k = Universe::_out_of_memory_error_java_heap->klass();
	    assert(k->klass_part()->name() == vmSymbols::java_lang_OutOfMemoryError(), "should be out of memory error");
	    k_h = instanceKlassHandle(THREAD, k);
	
	    int len = (StackTraceInThrowable) ? (int)PreallocatedOutOfMemoryErrorCount : 0;
	    Universe::_preallocated_out_of_memory_error_array = oopFactory::new_objArray(k_h(), len, CHECK_false);
	    for (int i=0; i<len; i++) {
	      oop err = k_h->allocate_permanent_instance(CHECK_false);
	      Handle err_h = Handle(THREAD, err);
	      java_lang_Throwable::allocate_backtrace(err_h, CHECK_false);
	      Universe::preallocated_out_of_memory_errors()->obj_at_put(i, err_h());
	    }
	    Universe::_preallocated_out_of_memory_error_avail_count = (jint)len;
	  }
	
	
  {- -------------------------------------------
  (1) Universe::_finalizer_register_cache フィールドに格納してある LatestMethodOopCache オブジェクトを初期化.
      ---------------------------------------- -}

	  // Setup static method for registering finalizers
	  // The finalizer klass must be linked before looking up the method, in
	  // case it needs to get rewritten.
	  instanceKlass::cast(SystemDictionary::Finalizer_klass())->link_class(CHECK_false);
	  methodOop m = instanceKlass::cast(SystemDictionary::Finalizer_klass())->find_method(
	                                  vmSymbols::register_method_name(),
	                                  vmSymbols::register_method_signature());
	  if (m == NULL || !m->is_static()) {
	    THROW_MSG_(vmSymbols::java_lang_NoSuchMethodException(),
	      "java.lang.ref.Finalizer.register", false);
	  }
	  Universe::_finalizer_register_cache->init(
	    SystemDictionary::Finalizer_klass(), m, CHECK_false);
	
  {- -------------------------------------------
  (1) Universe::_reflect_invoke_cache フィールドに格納してある ActiveMethodOopsCache オブジェクトを初期化.
      ---------------------------------------- -}

	  // Resolve on first use and initialize class.
	  // Note: No race-condition here, since a resolve will always return the same result
	
	  // Setup method for security checks
	  k = SystemDictionary::resolve_or_fail(vmSymbols::java_lang_reflect_Method(), true, CHECK_false);
	  k_h = instanceKlassHandle(THREAD, k);
	  k_h->link_class(CHECK_false);
	  m = k_h->find_method(vmSymbols::invoke_name(), vmSymbols::object_object_array_object_signature());
	  if (m == NULL || m->is_static()) {
	    THROW_MSG_(vmSymbols::java_lang_NoSuchMethodException(),
	      "java.lang.reflect.Method.invoke", false);
	  }
	  Universe::_reflect_invoke_cache->init(k_h(), m, CHECK_false);
	
  {- -------------------------------------------
  (1) Universe::_loader_addClass_cache フィールドに格納してある LatestMethodOopCache オブジェクトを初期化.
      ---------------------------------------- -}

	  // Setup method for registering loaded classes in class loader vector
	  instanceKlass::cast(SystemDictionary::ClassLoader_klass())->link_class(CHECK_false);
	  m = instanceKlass::cast(SystemDictionary::ClassLoader_klass())->find_method(vmSymbols::addClass_name(), vmSymbols::class_void_signature());
	  if (m == NULL || m->is_static()) {
	    THROW_MSG_(vmSymbols::java_lang_NoSuchMethodException(),
	      "java.lang.ClassLoader.addClass", false);
	  }
	  Universe::_loader_addClass_cache->init(
	    SystemDictionary::ClassLoader_klass(), m, CHECK_false);
	
  {- -------------------------------------------
  (1) initialize_converter_functions() を呼んで, 
      JVM_SetPrimitiveFieldValues()/JVM_GetPrimitiveFieldValues() 内で呼び出される補助関数をバインドする.
      (See: NativeLookup)
      ---------------------------------------- -}

	  // The folowing is initializing converter functions for serialization in
	  // JVM.cpp. If we clean up the StrictMath code above we may want to find
	  // a better solution for this as well.
	  initialize_converter_functions();
	
  {- -------------------------------------------
  (1) Universe::update_heap_info_at_gc() を呼んで, 
      以下の値を現在のヒープの最大長(capacity)及び使用量(used)の値で初期化しておく.
        * Universe::get_heap_capacity_at_last_gc()
        * Universe::get_heap_free_at_last_gc() 
        * Universe::get_heap_used_at_last_gc()
  
      なお, これらの値は ReferencePolicy オブジェクト内で soft reference の消去ポリシーの決定に使用されている
      (このため, 初回の GC が行われる前に初期化しておく必要がある).
      (See: ReferencePolicy)
      ---------------------------------------- -}

	  // This needs to be done before the first scavenge/gc, since
	  // it's an input to soft ref clearing policy.
	  {
	    MutexLocker x(Heap_lock);
	    Universe::update_heap_info_at_gc();
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // ("weak") refs processing infrastructure initialization
	  Universe::heap()->post_initialize();
	
  {- -------------------------------------------
  (1) 初期化が終わったので GC を解禁する.
      
      (なお, 対応する GC_locker::lock() は universe_init() 内で行われている)
      ---------------------------------------- -}

	  GC_locker::unlock();  // allow gc after bootstrapping
	
  {- -------------------------------------------
  (1) MemoryService::set_universe_heap() を呼んで
      MemoryManager オブジェクトおよび MemoryPool オブジェクトを初期化する.
      (See: [here](no211477i.html) for details)
      ---------------------------------------- -}

	  MemoryService::set_universe_heap(Universe::_collectedHeap);

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return true;
	}
	
```


