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
void Universe::genesis(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  ResourceMark rm;
	  { FlagSetting fs(_bootstrapping, true);
	
	    { MutexLocker mc(Compile_lock);
	
  {- -------------------------------------------
  (1) 各種 Klass オブジェクトの create_klass() メソッドを呼び出し, Klass オブジェクトを生成.
      (klassKlass, arrayKlassKlass, instanceKlassKlass, methodKlass, 等)
      ---------------------------------------- -}

	      // determine base vtable size; without that we cannot create the array klasses
	      compute_base_vtable_size();
	
	      if (!UseSharedSpaces) {
	        _klassKlassObj          = klassKlass::create_klass(CHECK);
	        _arrayKlassKlassObj     = arrayKlassKlass::create_klass(CHECK);
	
	        _objArrayKlassKlassObj  = objArrayKlassKlass::create_klass(CHECK);
	        _instanceKlassKlassObj  = instanceKlassKlass::create_klass(CHECK);
	        _typeArrayKlassKlassObj = typeArrayKlassKlass::create_klass(CHECK);
	
	        _boolArrayKlassObj      = typeArrayKlass::create_klass(T_BOOLEAN, sizeof(jboolean), CHECK);
	        _charArrayKlassObj      = typeArrayKlass::create_klass(T_CHAR,    sizeof(jchar),    CHECK);
	        _singleArrayKlassObj    = typeArrayKlass::create_klass(T_FLOAT,   sizeof(jfloat),   CHECK);
	        _doubleArrayKlassObj    = typeArrayKlass::create_klass(T_DOUBLE,  sizeof(jdouble),  CHECK);
	        _byteArrayKlassObj      = typeArrayKlass::create_klass(T_BYTE,    sizeof(jbyte),    CHECK);
	        _shortArrayKlassObj     = typeArrayKlass::create_klass(T_SHORT,   sizeof(jshort),   CHECK);
	        _intArrayKlassObj       = typeArrayKlass::create_klass(T_INT,     sizeof(jint),     CHECK);
	        _longArrayKlassObj      = typeArrayKlass::create_klass(T_LONG,    sizeof(jlong),    CHECK);
	
	        _typeArrayKlassObjs[T_BOOLEAN] = _boolArrayKlassObj;
	        _typeArrayKlassObjs[T_CHAR]    = _charArrayKlassObj;
	        _typeArrayKlassObjs[T_FLOAT]   = _singleArrayKlassObj;
	        _typeArrayKlassObjs[T_DOUBLE]  = _doubleArrayKlassObj;
	        _typeArrayKlassObjs[T_BYTE]    = _byteArrayKlassObj;
	        _typeArrayKlassObjs[T_SHORT]   = _shortArrayKlassObj;
	        _typeArrayKlassObjs[T_INT]     = _intArrayKlassObj;
	        _typeArrayKlassObjs[T_LONG]    = _longArrayKlassObj;
	
	        _methodKlassObj             = methodKlass::create_klass(CHECK);
	        _constMethodKlassObj        = constMethodKlass::create_klass(CHECK);
	        _methodDataKlassObj         = methodDataKlass::create_klass(CHECK);
	        _constantPoolKlassObj       = constantPoolKlass::create_klass(CHECK);
	        _constantPoolCacheKlassObj  = constantPoolCacheKlass::create_klass(CHECK);
	
	        _compiledICHolderKlassObj   = compiledICHolderKlass::create_klass(CHECK);
	        _systemObjArrayKlassObj     = objArrayKlassKlass::cast(objArrayKlassKlassObj())->allocate_system_objArray_klass(CHECK);
	
  {- -------------------------------------------
  (1) ??
      ---------------------------------------- -}

	        _the_empty_byte_array       = oopFactory::new_permanent_byteArray(0, CHECK);
	        _the_empty_short_array      = oopFactory::new_permanent_shortArray(0, CHECK);
	        _the_empty_int_array        = oopFactory::new_permanent_intArray(0, CHECK);
	        _the_empty_system_obj_array = oopFactory::new_system_objArray(0, CHECK);
	
	        _the_array_interfaces_array = oopFactory::new_system_objArray(2, CHECK);
	      }
	    }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    vmSymbols::initialize(CHECK);
	
	    SystemDictionary::initialize(CHECK);
	
	    klassOop ok = SystemDictionary::Object_klass();
	
	    _the_null_string            = StringTable::intern("null", CHECK);
	    _the_min_jint_string       = StringTable::intern("-2147483648", CHECK);
	
	    if (UseSharedSpaces) {
	      // Verify shared interfaces array.
	      assert(_the_array_interfaces_array->obj_at(0) ==
	             SystemDictionary::Cloneable_klass(), "u3");
	      assert(_the_array_interfaces_array->obj_at(1) ==
	             SystemDictionary::Serializable_klass(), "u3");
	
	      // Verify element klass for system obj array klass
	      assert(objArrayKlass::cast(_systemObjArrayKlassObj)->element_klass() == ok, "u1");
	      assert(objArrayKlass::cast(_systemObjArrayKlassObj)->bottom_klass() == ok, "u2");
	
	      // Verify super class for the classes created above
	      assert(Klass::cast(boolArrayKlassObj()     )->super() == ok, "u3");
	      assert(Klass::cast(charArrayKlassObj()     )->super() == ok, "u3");
	      assert(Klass::cast(singleArrayKlassObj()   )->super() == ok, "u3");
	      assert(Klass::cast(doubleArrayKlassObj()   )->super() == ok, "u3");
	      assert(Klass::cast(byteArrayKlassObj()     )->super() == ok, "u3");
	      assert(Klass::cast(shortArrayKlassObj()    )->super() == ok, "u3");
	      assert(Klass::cast(intArrayKlassObj()      )->super() == ok, "u3");
	      assert(Klass::cast(longArrayKlassObj()     )->super() == ok, "u3");
	      assert(Klass::cast(constantPoolKlassObj()  )->super() == ok, "u3");
	      assert(Klass::cast(systemObjArrayKlassObj())->super() == ok, "u3");
	    } else {
	      // Set up shared interfaces array.  (Do this before supers are set up.)
	      _the_array_interfaces_array->obj_at_put(0, SystemDictionary::Cloneable_klass());
	      _the_array_interfaces_array->obj_at_put(1, SystemDictionary::Serializable_klass());
	
	      // Set element klass for system obj array klass
	      objArrayKlass::cast(_systemObjArrayKlassObj)->set_element_klass(ok);
	      objArrayKlass::cast(_systemObjArrayKlassObj)->set_bottom_klass(ok);
	
	      // Set super class for the classes created above
	      Klass::cast(boolArrayKlassObj()     )->initialize_supers(ok, CHECK);
	      Klass::cast(charArrayKlassObj()     )->initialize_supers(ok, CHECK);
	      Klass::cast(singleArrayKlassObj()   )->initialize_supers(ok, CHECK);
	      Klass::cast(doubleArrayKlassObj()   )->initialize_supers(ok, CHECK);
	      Klass::cast(byteArrayKlassObj()     )->initialize_supers(ok, CHECK);
	      Klass::cast(shortArrayKlassObj()    )->initialize_supers(ok, CHECK);
	      Klass::cast(intArrayKlassObj()      )->initialize_supers(ok, CHECK);
	      Klass::cast(longArrayKlassObj()     )->initialize_supers(ok, CHECK);
	      Klass::cast(constantPoolKlassObj()  )->initialize_supers(ok, CHECK);
	      Klass::cast(systemObjArrayKlassObj())->initialize_supers(ok, CHECK);
	      Klass::cast(boolArrayKlassObj()     )->set_super(ok);
	      Klass::cast(charArrayKlassObj()     )->set_super(ok);
	      Klass::cast(singleArrayKlassObj()   )->set_super(ok);
	      Klass::cast(doubleArrayKlassObj()   )->set_super(ok);
	      Klass::cast(byteArrayKlassObj()     )->set_super(ok);
	      Klass::cast(shortArrayKlassObj()    )->set_super(ok);
	      Klass::cast(intArrayKlassObj()      )->set_super(ok);
	      Klass::cast(longArrayKlassObj()     )->set_super(ok);
	      Klass::cast(constantPoolKlassObj()  )->set_super(ok);
	      Klass::cast(systemObjArrayKlassObj())->set_super(ok);
	    }
	
	    Klass::cast(boolArrayKlassObj()     )->append_to_sibling_list();
	    Klass::cast(charArrayKlassObj()     )->append_to_sibling_list();
	    Klass::cast(singleArrayKlassObj()   )->append_to_sibling_list();
	    Klass::cast(doubleArrayKlassObj()   )->append_to_sibling_list();
	    Klass::cast(byteArrayKlassObj()     )->append_to_sibling_list();
	    Klass::cast(shortArrayKlassObj()    )->append_to_sibling_list();
	    Klass::cast(intArrayKlassObj()      )->append_to_sibling_list();
	    Klass::cast(longArrayKlassObj()     )->append_to_sibling_list();
	    Klass::cast(constantPoolKlassObj()  )->append_to_sibling_list();
	    Klass::cast(systemObjArrayKlassObj())->append_to_sibling_list();
	  } // end of core bootstrapping
	
	  // Initialize _objectArrayKlass after core bootstraping to make
	  // sure the super class is set up properly for _objectArrayKlass.
	  _objectArrayKlassObj = instanceKlass::
	    cast(SystemDictionary::Object_klass())->array_klass(1, CHECK);
	  // Add the class to the class hierarchy manually to make sure that
	  // its vtable is initialized after core bootstrapping is completed.
	  Klass::cast(_objectArrayKlassObj)->append_to_sibling_list();
	
  {- -------------------------------------------
  (1) もし JDK_Version が完全には分かっていなかった場合には, 
      使用している JDK のバージョンを推定し, 
      JDK_Version::fully_initialize() で JDK_Version に記録しておく.
    
      なお, 推定は以下のように行っている.
      * java_lang_management_MemoryUsage を持っていれば, 1.5
      * (上記のクラスがなく) java_lang_CharSequence を持っていれば, 1.4
      * (上記のクラスがなく) java_lang_Shutdown を持っていれば, 1.3
      * 上記のクラスをどれも持っていなければ,  1.2
      ---------------------------------------- -}

	  // Compute is_jdk version flags.
	  // Only 1.3 or later has the java.lang.Shutdown class.
	  // Only 1.4 or later has the java.lang.CharSequence interface.
	  // Only 1.5 or later has the java.lang.management.MemoryUsage class.
	  if (JDK_Version::is_partially_initialized()) {
	    uint8_t jdk_version;
	    klassOop k = SystemDictionary::resolve_or_null(
	        vmSymbols::java_lang_management_MemoryUsage(), THREAD);
	    CLEAR_PENDING_EXCEPTION; // ignore exceptions
	    if (k == NULL) {
	      k = SystemDictionary::resolve_or_null(
	          vmSymbols::java_lang_CharSequence(), THREAD);
	      CLEAR_PENDING_EXCEPTION; // ignore exceptions
	      if (k == NULL) {
	        k = SystemDictionary::resolve_or_null(
	            vmSymbols::java_lang_Shutdown(), THREAD);
	        CLEAR_PENDING_EXCEPTION; // ignore exceptions
	        if (k == NULL) {
	          jdk_version = 2;
	        } else {
	          jdk_version = 3;
	        }
	      } else {
	        jdk_version = 4;
	      }
	    } else {
	      jdk_version = 5;
	    }
	    JDK_Version::fully_initialize(jdk_version);
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	  #ifdef ASSERT
	  if (FullGCALot) {
	    // Allocate an array of dummy objects.
	    // We'd like these to be at the bottom of the old generation,
	    // so that when we free one and then collect,
	    // (almost) the whole heap moves
	    // and we find out if we actually update all the oops correctly.
	    // But we can't allocate directly in the old generation,
	    // so we allocate wherever, and hope that the first collection
	    // moves these objects to the bottom of the old generation.
	    // We can allocate directly in the permanent generation, so we do.
	    int size;
	    if (UseConcMarkSweepGC) {
	      warning("Using +FullGCALot with concurrent mark sweep gc "
	              "will not force all objects to relocate");
	      size = FullGCALotDummies;
	    } else {
	      size = FullGCALotDummies * 2;
	    }
	    objArrayOop    naked_array = oopFactory::new_system_objArray(size, CHECK);
	    objArrayHandle dummy_array(THREAD, naked_array);
	    int i = 0;
	    while (i < size) {
	      if (!UseConcMarkSweepGC) {
	        // Allocate dummy in old generation
	        oop dummy = instanceKlass::cast(SystemDictionary::Object_klass())->allocate_instance(CHECK);
	        dummy_array->obj_at_put(i++, dummy);
	      }
	      // Allocate dummy in permanent generation
	      oop dummy = instanceKlass::cast(SystemDictionary::Object_klass())->allocate_permanent_instance(CHECK);
	      dummy_array->obj_at_put(i++, dummy);
	    }
	    {
	      // Only modify the global variable inside the mutex.
	      // If we had a race to here, the other dummy_array instances
	      // and their elements just get dropped on the floor, which is fine.
	      MutexLocker ml(FullGCALot_lock);
	      if (_fullgc_alot_dummy_array == NULL) {
	        _fullgc_alot_dummy_array = dummy_array();
	      }
	    }
	    assert(i == _fullgc_alot_dummy_array->length(), "just checking");
	  }
	  #endif
	}
	
```


