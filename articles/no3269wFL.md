---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp

### 名前(function name)
```
void SystemDictionary::define_instance_class(instanceKlassHandle k, TRAPS) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle class_loader_h(THREAD, k->class_loader());
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	 // for bootstrap and other parallel classloaders don't acquire lock,
	 // use placeholder token
	 // If a parallelCapable class loader calls define_instance_class instead of
	 // find_or_define_instance_class to get here, we have a timing
	 // hole with systemDictionary updates and check_constraints
	 if (!class_loader_h.is_null() && !is_parallelCapable(class_loader_h)) {
	    assert(ObjectSynchronizer::current_thread_holds_lock((JavaThread*)THREAD,
	         compute_loader_lock_object(class_loader_h, THREAD)),
	         "define called without lock");
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Check class-loading constraints. Throw exception if violation is detected.
	  // Grabs and releases SystemDictionary_lock
	  // The check_constraints/find_class call and update_dictionary sequence
	  // must be "atomic" for a specific class/classloader pair so we never
	  // define two different instanceKlasses for that class/classloader pair.
	  // Existing classloaders will call define_instance_class with the
	  // classloader lock held
	  // Parallel classloaders will call find_or_define_instance_class
	  // which will require a token to perform the define class
	  Symbol*  name_h = k->name();
	  unsigned int d_hash = dictionary()->compute_hash(name_h, class_loader_h);
	  int d_index = dictionary()->hash_to_index(d_hash);
	  check_constraints(d_index, d_hash, k, class_loader_h, true, CHECK);
	
  {- -------------------------------------------
  (1) 処理対象のクラスがシステムクラスでない場合は (= instanceKlass::class_loader() の返値が NULL でなければ), 
      JavaCalls::call() で java.lang.ClassLoader.addClass() メソッドを呼び出して
      クラスローダーに登録しておく.
  
      (なおこの処理は SystemDictionary に登録する前に行っている.
       理由は, この処理の途中で OutOfMemoryError が生じる可能性が有り, 
       その場合は SystemDictionary に登録されないようにするため)
      ---------------------------------------- -}

	  // Register class just loaded with class loader (placed in Vector)
	  // Note we do this before updating the dictionary, as this can
	  // fail with an OutOfMemoryError (if it does, we will *not* put this
	  // class in the dictionary and will not update the class hierarchy).
	  if (k->class_loader() != NULL) {
	    methodHandle m(THREAD, Universe::loader_addClass_method());
	    JavaValue result(T_VOID);
	    JavaCallArguments args(class_loader_h);
	    args.push_oop(Handle(THREAD, k->java_mirror()));
	    JavaCalls::call(&result, m, &args, CHECK);
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Add the new class. We need recompile lock during update of CHA.
	  {
	    unsigned int p_hash = placeholders()->compute_hash(name_h, class_loader_h);
	    int p_index = placeholders()->hash_to_index(p_hash);
	
	    MutexLocker mu_r(Compile_lock, THREAD);
	
	    // Add to class hierarchy, initialize vtables, and do possible
	    // deoptimizations.
	    add_to_hierarchy(k, CHECK); // No exception, but can block
	
	    // Add to systemDictionary - so other classes can see it.
	    // Grabs and releases SystemDictionary_lock
	    update_dictionary(d_index, d_hash, p_index, p_hash,
	                      k, class_loader_h, THREAD);
	  }
	  k->eager_initialize(THREAD);
	
  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  // notify jvmti
	  if (JvmtiExport::should_post_class_load()) {
	      assert(THREAD->is_Java_thread(), "thread->is_Java_thread()");
	      JvmtiExport::post_class_load((JavaThread *) THREAD, k());
	
	  }
	}
	
```


