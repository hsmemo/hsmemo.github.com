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
instanceKlassHandle SystemDictionary::load_instance_class(Symbol* class_name, Handle class_loader, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  instanceKlassHandle nh = instanceKlassHandle(); // null Handle

  {- -------------------------------------------
  (1) (以降の処理は, 使用する ClassLoader が
      指定されているかどうか (= classloader 引数が null かどうか) で 2つに分岐)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) 以下は, 使用する ClassLoader が指定されていない場合 (= classloader 引数が null の場合).
      この場合は, HotSpot 内蔵のクラスローダ(ブートストラップ・クラスローダ)でロードを行う.
      ---------------------------------------- -}

	  if (class_loader.is_null()) {
	
    {- -------------------------------------------
  (1.1) まず SystemDictionary::load_shared_class() を呼んで, 
        shared archive (Class Data Sharing(CDS) の領域) からロードを試みる.
  
        なお, (プロファイル情報の記録) も行っている. ("sun.cls.sharedClassLoadTime")
        (See: PerfTraceTime, ClassLoader::perf_shared_classload_time())
        ---------------------------------------- -}

	    // Search the shared system dictionary for classes preloaded into the
	    // shared spaces.
	    instanceKlassHandle k;
	    {
	      PerfTraceTime vmtimer(ClassLoader::perf_shared_classload_time());
	      k = load_shared_class(class_name, class_loader, THREAD);
	    }
	
    {- -------------------------------------------
  (1.1) クラスが見つからなければ, ClassLoader::load_classfile() を呼んで
        HotSpot 内蔵のクラスローダ(ブートストラップ・クラスローダ)でのロードを試みる.
  
        なお, (プロファイル情報の記録) も行っている. ("sun.cls.sysClassLoadTime")
        (See: PerfTraceTime, ClassLoader::perf_shared_classload_time())
        ---------------------------------------- -}

	    if (k.is_null()) {
	      // Use VM class loader
	      PerfTraceTime vmtimer(ClassLoader::perf_sys_classload_time());
	      k = ClassLoader::load_classfile(class_name, CHECK_(nh));
	    }
	
    {- -------------------------------------------
  (1.1) #TODO
        (???用の処理) (#ifdef KERNEL 時にのみ実行)
        (まだ見つかっていなければ,  sun.jkernel.DownloadManager.getBootClassPathEntryForClass() で
        ダウンロードする模様)
        ---------------------------------------- -}

	#ifdef KERNEL
	    // If the VM class loader has failed to load the class, call the
	    // DownloadManager class to make it magically appear on the classpath
	    // and try again.  This is only configured with the Kernel VM.
	    if (k.is_null()) {
	      k = download_and_retry_class_load(class_name, CHECK_(nh));
	    }
	#endif // KERNEL
	
    {- -------------------------------------------
  (1.1) 以上の処理でクラスが見つかった場合は, 
        SystemDictionary::find_or_define_instance_class() を呼んで
        SystemDictionary に登録しておく.
        ---------------------------------------- -}

	    // find_or_define_instance_class may return a different instanceKlass
	    if (!k.is_null()) {
	      k = find_or_define_instance_class(class_name, class_loader, k, CHECK_(nh));
	    }

    {- -------------------------------------------
  (1.1) 結果をリターン
        ---------------------------------------- -}

	    return k;

  {- -------------------------------------------
  (1) 以下は, 使用する ClassLoader が指定されている場合 (= classloader 引数が null ではない場合).
      この場合は, 指定された ClassLoader でロードを行う.
      ---------------------------------------- -}

	  } else {
	    // Use user specified class loader to load class. Call loadClass operation on class_loader.

    {- -------------------------------------------
  (1.1) (ResourceMark)
        ---------------------------------------- -}

	    ResourceMark rm(THREAD);
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    assert(THREAD->is_Java_thread(), "must be a JavaThread");

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    JavaThread* jt = (JavaThread*) THREAD;
	
    {- -------------------------------------------
  (1.1) (プロファイル情報の記録) ("sun.cls.appClassLoadTime", "sun.cls.appClassLoadTime.self", "sun.cls.appClassLoadCount") 
        (See: PerfClassTraceTime, ClassLoader::perf_app_classload_time(), ClassLoader::perf_app_classload_selftime(), ClassLoader::perf_app_classload_count())
        ---------------------------------------- -}

	    PerfClassTraceTime vmtimer(ClassLoader::perf_app_classload_time(),
	                               ClassLoader::perf_app_classload_selftime(),
	                               ClassLoader::perf_app_classload_count(),
	                               jt->get_thread_stat()->perf_recursion_counts_addr(),
	                               jt->get_thread_stat()->perf_timers_addr(),
	                               PerfClassTraceTime::CLASS_LOAD);
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    Handle s = java_lang_String::create_from_symbol(class_name, CHECK_(nh));
	    // Translate to external class name format, i.e., convert '/' chars to '.'
	    Handle string = java_lang_String::externalize_classname(s, CHECK_(nh));
	
	    JavaValue result(T_OBJECT);
	
	    KlassHandle spec_klass (THREAD, SystemDictionary::ClassLoader_klass());
	
    {- -------------------------------------------
  (1.1) classloader 引数で指定された ClassLoader オブジェクトの
        loadClassInternal() もしくは loadClass() を呼び出してロードを行う.
  
        呼び出すメソッドは, 以下の両方が成り立てば loadClassInternal(). そうでなければ loadClass().
        * MustCallLoadClassInternal オプションが指定されている.
        * ClassLoader.loadClassInternal() が存在する.
        ---------------------------------------- -}

	    // Call public unsynchronized loadClass(String) directly for all class loaders
	    // for parallelCapable class loaders. JDK >=7, loadClass(String, boolean) will
	    // acquire a class-name based lock rather than the class loader object lock.
	    // JDK < 7 already acquire the class loader lock in loadClass(String, boolean),
	    // so the call to loadClassInternal() was not required.
	    //
	    // UnsyncloadClass flag means both call loadClass(String) and do
	    // not acquire the class loader lock even for class loaders that are
	    // not parallelCapable. This was a risky transitional
	    // flag for diagnostic purposes only. It is risky to call
	    // custom class loaders without synchronization.
	    // WARNING If a custom class loader does NOT synchronizer findClass, or callers of
	    // findClass, the UnsyncloadClass flag risks unexpected timing bugs in the field.
	    // Do NOT assume this will be supported in future releases.
	    //
	    // Added MustCallLoadClassInternal in case we discover in the field
	    // a customer that counts on this call
	    if (MustCallLoadClassInternal && has_loadClassInternal()) {
	      JavaCalls::call_special(&result,
	                              class_loader,
	                              spec_klass,
	                              vmSymbols::loadClassInternal_name(),
	                              vmSymbols::string_class_signature(),
	                              string,
	                              CHECK_(nh));
	    } else {
	      JavaCalls::call_virtual(&result,
	                              class_loader,
	                              spec_klass,
	                              vmSymbols::loadClass_name(),
	                              vmSymbols::string_class_signature(),
	                              string,
	                              CHECK_(nh));
	    }
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    assert(result.get_type() == T_OBJECT, "just checking");
	    oop obj = (oop) result.get_jobject();
	
    {- -------------------------------------------
  (1.1) クラスのロードに成功しており, かつプリミティブ型のクラスではなく, 
        かつクラス名が class_name 引数と一致する場合は, 
        結果をリターンする.
        ---------------------------------------- -}

	    // Primitive classes return null since forName() can not be
	    // used to obtain any of the Class objects representing primitives or void
	    if ((obj != NULL) && !(java_lang_Class::is_primitive(obj))) {
	      instanceKlassHandle k =
	                instanceKlassHandle(THREAD, java_lang_Class::as_klassOop(obj));
	      // For user defined Java class loaders, check that the name returned is
	      // the same as that requested.  This check is done for the bootstrap
	      // loader when parsing the class file.
	      if (class_name == k->name()) {
	        return k;
	      }
	    }

    {- -------------------------------------------
  (1.1) (ここに来るのはクラスが見つからなかった場合)
        null Handle をリターン
        ---------------------------------------- -}

	    // Class is not found or has the wrong name, return NULL
	    return nh;
	  }
	}
	
```


