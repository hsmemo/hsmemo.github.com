---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp
### 説明(description)

```
// Install the redefinition of a class:
//    - house keeping (flushing breakpoints and caches, deoptimizing
//      dependent compiled code)
//    - replacing parts in the_class with parts from scratch_class
//    - adding a weak reference to track the obsolete but interesting
//      parts of the_class
//    - adjusting constant pool caches and vtables in other classes
//      that refer to methods in the_class. These adjustments use the
//      SystemDictionary::classes_do() facility which only allows
//      a helper method to be specified. The interesting parameters
//      that we would like to pass to the helper method are saved in
//      static global fields in the VM operation.
```

### 名前(function name)
```
void VM_RedefineClasses::redefine_single_class(jclass the_jclass,
       instanceKlassHandle scratch_class, TRAPS) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  RC_TIMER_START(_timer_rsc_phase1);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  oop the_class_mirror = JNIHandles::resolve_non_null(the_jclass);
	  klassOop the_class_oop = java_lang_Class::as_klassOop(the_class_mirror);
	  instanceKlassHandle the_class = instanceKlassHandle(THREAD, the_class_oop);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	#ifndef JVMTI_KERNEL
	  // Remove all breakpoints in methods of this class
	  JvmtiBreakpoints& jvmti_breakpoints = JvmtiCurrentBreakpoints::get_jvmti_breakpoints();
	  jvmti_breakpoints.clearall_in_class_at_safepoint(the_class_oop);
	#endif // !JVMTI_KERNEL
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (the_class_oop == Universe::reflect_invoke_cache()->klass()) {
	    // We are redefining java.lang.reflect.Method. Method.invoke() is
	    // cached and users of the cache care about each active version of
	    // the method so we have to track this previous version.
	    // Do this before methods get switched
	    Universe::reflect_invoke_cache()->add_previous_version(
	      the_class->method_with_idnum(Universe::reflect_invoke_cache()->method_idnum()));
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Deoptimize all compiled code that depends on this class
	  flush_dependent_code(the_class, THREAD);
	
	  _old_methods = the_class->methods();
	  _new_methods = scratch_class->methods();
	  _the_class_oop = the_class_oop;
	  compute_added_deleted_matching_methods();
	  update_jmethod_ids();
	
	  // Attach new constant pool to the original klass. The original
	  // klass still refers to the old constant pool (for now).
	  scratch_class->constants()->set_pool_holder(the_class());
	
  {- -------------------------------------------
  (1) (以下のコードは #if 0 でコメントアウトされているが... #TODO)
      ---------------------------------------- -}

	#if 0
	  // In theory, with constant pool merging in place we should be able
	  // to save space by using the new, merged constant pool in place of
	  // the old constant pool(s). By "pool(s)" I mean the constant pool in
	  // the klass version we are replacing now and any constant pool(s) in
	  // previous versions of klass. Nice theory, doesn't work in practice.
	  // When this code is enabled, even simple programs throw NullPointer
	  // exceptions. I'm guessing that this is caused by some constant pool
	  // cache difference between the new, merged constant pool and the
	  // constant pool that was just being used by the klass. I'm keeping
	  // this code around to archive the idea, but the code has to remain
	  // disabled for now.
	
	  // Attach each old method to the new constant pool. This can be
	  // done here since we are past the bytecode verification and
	  // constant pool optimization phases.
	  for (int i = _old_methods->length() - 1; i >= 0; i--) {
	    methodOop method = (methodOop)_old_methods->obj_at(i);
	    method->set_constants(scratch_class->constants());
	  }
	
	  {
	    // walk all previous versions of the klass
	    instanceKlass *ik = (instanceKlass *)the_class()->klass_part();
	    PreviousVersionWalker pvw(ik);
	    instanceKlassHandle ikh;
	    do {
	      ikh = pvw.next_previous_version();
	      if (!ikh.is_null()) {
	        ik = ikh();
	
	        // attach previous version of klass to the new constant pool
	        ik->set_constants(scratch_class->constants());
	
	        // Attach each method in the previous version of klass to the
	        // new constant pool
	        objArrayOop prev_methods = ik->methods();
	        for (int i = prev_methods->length() - 1; i >= 0; i--) {
	          methodOop method = (methodOop)prev_methods->obj_at(i);
	          method->set_constants(scratch_class->constants());
	        }
	      }
	    } while (!ikh.is_null());
	  }
	#endif
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Replace methods and constantpool
	  the_class->set_methods(_new_methods);
	  scratch_class->set_methods(_old_methods);     // To prevent potential GCing of the old methods,
	                                          // and to be able to undo operation easily.
	
	  constantPoolOop old_constants = the_class->constants();
	  the_class->set_constants(scratch_class->constants());
	  scratch_class->set_constants(old_constants);  // See the previous comment.

  {- -------------------------------------------
  (1) (以下のコードは #if 0 でコメントアウトされているが... #TODO)
      ---------------------------------------- -}

	#if 0
	  // We are swapping the guts of "the new class" with the guts of "the
	  // class". Since the old constant pool has just been attached to "the
	  // new class", it seems logical to set the pool holder in the old
	  // constant pool also. However, doing this will change the observable
	  // class hierarchy for any old methods that are still executing. A
	  // method can query the identity of its "holder" and this query uses
	  // the method's constant pool link to find the holder. The change in
	  // holding class from "the class" to "the new class" can confuse
	  // things.
	  //
	  // Setting the old constant pool's holder will also cause
	  // verification done during vtable initialization below to fail.
	  // During vtable initialization, the vtable's class is verified to be
	  // a subtype of the method's holder. The vtable's class is "the
	  // class" and the method's holder is gotten from the constant pool
	  // link in the method itself. For "the class"'s directly implemented
	  // methods, the method holder is "the class" itself (as gotten from
	  // the new constant pool). The check works fine in this case. The
	  // check also works fine for methods inherited from super classes.
	  //
	  // Miranda methods are a little more complicated. A miranda method is
	  // provided by an interface when the class implementing the interface
	  // does not provide its own method.  These interfaces are implemented
	  // internally as an instanceKlass. These special instanceKlasses
	  // share the constant pool of the class that "implements" the
	  // interface. By sharing the constant pool, the method holder of a
	  // miranda method is the class that "implements" the interface. In a
	  // non-redefine situation, the subtype check works fine. However, if
	  // the old constant pool's pool holder is modified, then the check
	  // fails because there is no class hierarchy relationship between the
	  // vtable's class and "the new class".
	
	  old_constants->set_pool_holder(scratch_class());
	#endif
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // track which methods are EMCP for add_previous_version() call below
	  BitMap emcp_methods(_old_methods->length());
	  int emcp_method_count = 0;
	  emcp_methods.clear();  // clears 0..(length() - 1)
	  check_methods_and_mark_as_obsolete(&emcp_methods, &emcp_method_count);
	  transfer_old_native_function_registrations(the_class);
	
	  // The class file bytes from before any retransformable agents mucked
	  // with them was cached on the scratch class, move to the_class.
	  // Note: we still want to do this if nothing needed caching since it
	  // should get cleared in the_class too.
	  the_class->set_cached_class_file(scratch_class->get_cached_class_file_bytes(),
	                                   scratch_class->get_cached_class_file_len());
	
	  // Replace inner_classes
	  typeArrayOop old_inner_classes = the_class->inner_classes();
	  the_class->set_inner_classes(scratch_class->inner_classes());
	  scratch_class->set_inner_classes(old_inner_classes);
	
	  // Initialize the vtable and interface table after
	  // methods have been rewritten
	  {
	    ResourceMark rm(THREAD);
	    // no exception should happen here since we explicitly
	    // do not check loader constraints.
	    // compare_and_normalize_class_versions has already checked:
	    //  - classloaders unchanged, signatures unchanged
	    //  - all instanceKlasses for redefined classes reused & contents updated
	    the_class->vtable()->initialize_vtable(false, THREAD);
	    the_class->itable()->initialize_itable(false, THREAD);
	    assert(!HAS_PENDING_EXCEPTION || (THREAD->pending_exception()->is_a(SystemDictionary::ThreadDeath_klass())), "redefine exception");
	  }
	
	  // Leave arrays of jmethodIDs and itable index cache unchanged
	
	  // Copy the "source file name" attribute from new class version
	  the_class->set_source_file_name(scratch_class->source_file_name());
	
	  // Copy the "source debug extension" attribute from new class version
	  the_class->set_source_debug_extension(
	    scratch_class->source_debug_extension());
	
	  // Use of javac -g could be different in the old and the new
	  if (scratch_class->access_flags().has_localvariable_table() !=
	      the_class->access_flags().has_localvariable_table()) {
	
	    AccessFlags flags = the_class->access_flags();
	    if (scratch_class->access_flags().has_localvariable_table()) {
	      flags.set_has_localvariable_table();
	    } else {
	      flags.clear_has_localvariable_table();
	    }
	    the_class->set_access_flags(flags);
	  }
	
	  // Replace class annotation fields values
	  typeArrayOop old_class_annotations = the_class->class_annotations();
	  the_class->set_class_annotations(scratch_class->class_annotations());
	  scratch_class->set_class_annotations(old_class_annotations);
	
	  // Replace fields annotation fields values
	  objArrayOop old_fields_annotations = the_class->fields_annotations();
	  the_class->set_fields_annotations(scratch_class->fields_annotations());
	  scratch_class->set_fields_annotations(old_fields_annotations);
	
	  // Replace methods annotation fields values
	  objArrayOop old_methods_annotations = the_class->methods_annotations();
	  the_class->set_methods_annotations(scratch_class->methods_annotations());
	  scratch_class->set_methods_annotations(old_methods_annotations);
	
	  // Replace methods parameter annotation fields values
	  objArrayOop old_methods_parameter_annotations =
	    the_class->methods_parameter_annotations();
	  the_class->set_methods_parameter_annotations(
	    scratch_class->methods_parameter_annotations());
	  scratch_class->set_methods_parameter_annotations(old_methods_parameter_annotations);
	
	  // Replace methods default annotation fields values
	  objArrayOop old_methods_default_annotations =
	    the_class->methods_default_annotations();
	  the_class->set_methods_default_annotations(
	    scratch_class->methods_default_annotations());
	  scratch_class->set_methods_default_annotations(old_methods_default_annotations);
	
	  // Replace minor version number of class file
	  u2 old_minor_version = the_class->minor_version();
	  the_class->set_minor_version(scratch_class->minor_version());
	  scratch_class->set_minor_version(old_minor_version);
	
	  // Replace major version number of class file
	  u2 old_major_version = the_class->major_version();
	  the_class->set_major_version(scratch_class->major_version());
	  scratch_class->set_major_version(old_major_version);
	
	  // Replace CP indexes for class and name+type of enclosing method
	  u2 old_class_idx  = the_class->enclosing_method_class_index();
	  u2 old_method_idx = the_class->enclosing_method_method_index();
	  the_class->set_enclosing_method_indices(
	    scratch_class->enclosing_method_class_index(),
	    scratch_class->enclosing_method_method_index());
	  scratch_class->set_enclosing_method_indices(old_class_idx, old_method_idx);
	
	  // keep track of previous versions of this class
	  the_class->add_previous_version(scratch_class, &emcp_methods,
	    emcp_method_count);
	
	  RC_TIMER_STOP(_timer_rsc_phase1);
	  RC_TIMER_START(_timer_rsc_phase2);
	
	  // Adjust constantpool caches and vtables for all classes
	  // that reference methods of the evolved class.
	  SystemDictionary::classes_do(adjust_cpool_cache_and_vtable, THREAD);
	
	  if (the_class->oop_map_cache() != NULL) {
	    // Flush references to any obsolete methods from the oop map cache
	    // so that obsolete methods are not pinned.
	    the_class->oop_map_cache()->flush_obsolete_entries();
	  }
	
	  // increment the classRedefinedCount field in the_class and in any
	  // direct and indirect subclasses of the_class
	  increment_class_counter((instanceKlass *)the_class()->klass_part(), THREAD);
	
	  // RC_TRACE macro has an embedded ResourceMark
	  RC_TRACE_WITH_THREAD(0x00000001, THREAD,
	    ("redefined name=%s, count=%d (avail_mem=" UINT64_FORMAT "K)",
	    the_class->external_name(),
	    java_lang_Class::classRedefinedCount(the_class_mirror),
	    os::available_memory() >> 10));
	
	  RC_TIMER_STOP(_timer_rsc_phase2);
	} // end redefine_single_class()
	
```


