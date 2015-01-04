---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp
### 説明(description)

```
// Add a klass to the system from a stream (called by jni_DefineClass and
// JVM_DefineClass).
// Note: class_name can be NULL. In that case we do not know the name of
// the class until we have parsed the stream.

```

### 名前(function name)
```
klassOop SystemDictionary::resolve_from_stream(Symbol* class_name,
                                               Handle class_loader,
                                               Handle protection_domain,
                                               ClassFileStream* st,
                                               bool verify,
                                               TRAPS) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (DoObjectLock は, ロックを取得するかどうかを示す. parallel に処理可能なクラスローダーなら取得しない)
      ---------------------------------------- -}

	  // Classloaders that support parallelism, e.g. bootstrap classloader,
	  // or all classloaders with UnsyncloadClass do not acquire lock here
	  bool DoObjectLock = true;
	  if (is_parallelCapable(class_loader)) {
	    DoObjectLock = false;
	  }
	
  {- -------------------------------------------
  (1) (以降の処理は ... で排他した状態で行う)
  
      なお, (プロファイル情報の記録) も行っている. ("sun.cls.systemLoaderLockContentionRate", "sun.cls.nonSystemLoaderLockContentionRate")
      (See: SystemDictionary::check_loader_lock_contention())
      ---------------------------------------- -}

	  // Make sure we are synchronized on the class loader before we proceed
	  Handle lockObject = compute_loader_lock_object(class_loader, THREAD);
	  check_loader_lock_contention(lockObject, THREAD);
	  ObjectLocker ol(lockObject, THREAD, DoObjectLock);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  TempNewSymbol parsed_name = NULL;
	
  {- -------------------------------------------
  (1) ClassFileParser::parseClassFile() を呼んで, クラスのパース処理を行う
      ---------------------------------------- -}

	  // Parse the stream. Note that we do this even though this klass might
	  // already be present in the SystemDictionary, otherwise we would not
	  // throw potential ClassFormatErrors.
	  //
	  // Note: "name" is updated.
	  // Further note:  a placeholder will be added for this class when
	  //   super classes are loaded (resolve_super_or_fail). We expect this
	  //   to be called for all classes but java.lang.Object; and we preload
	  //   java.lang.Object through resolve_or_fail, not this path.
	
	  instanceKlassHandle k = ClassFileParser(st).parseClassFile(class_name,
	                                                             class_loader,
	                                                             protection_domain,
	                                                             parsed_name,
	                                                             verify,
	                                                             THREAD);
	
  {- -------------------------------------------
  (1) もしロードしたクラスが "java." パッケージに属していた場合, 
      ("java." パッケージへの定義は許可していないので) SecurityException を送出.
  
      (ただし, もう既に例外が出ている場合は, この処理は行わずに先に進む)
      ---------------------------------------- -}

	  const char* pkg = "java/";
	  if (!HAS_PENDING_EXCEPTION &&
	      !class_loader.is_null() &&
	      parsed_name != NULL &&
	      !strncmp((const char*)parsed_name->bytes(), pkg, strlen(pkg))) {
	    // It is illegal to define classes in the "java." package from
	    // JVM_DefineClass or jni_DefineClass unless you're the bootclassloader
	    ResourceMark rm(THREAD);
	    char* name = parsed_name->as_C_string();
	    char* index = strrchr(name, '/');
	    *index = '\0'; // chop to just the package name
	    while ((index = strchr(name, '/')) != NULL) {
	      *index = '.'; // replace '/' with '.' in package name
	    }
	    const char* fmt = "Prohibited package name: %s";
	    size_t len = strlen(fmt) + strlen(name);
	    char* message = NEW_RESOURCE_ARRAY(char, len);
	    jio_snprintf(message, len, fmt, name);
	    Exceptions::_throw_msg(THREAD_AND_LOCATION,
	      vmSymbols::java_lang_SecurityException(), message);
	  }
	
  {- -------------------------------------------
  (1) 以下のどちらかを呼んで SystemDictionary への登録を行う.
  
      * 並行に処理できる場合 (= SystemDictionary::is_parallelCapable() が true の場合):
        SystemDictionary::find_or_define_instance_class()
      * 〃できない場合:
        SystemDictionary::define_instance_class()
  
      (ただし, もう既に例外が出ている場合は, この処理は行わずに先に進む)
      ---------------------------------------- -}

	  if (!HAS_PENDING_EXCEPTION) {
	    assert(parsed_name != NULL, "Sanity");
	    assert(class_name == NULL || class_name == parsed_name, "name mismatch");
	    // Verification prevents us from creating names with dots in them, this
	    // asserts that that's the case.
	    assert(is_internal_format(parsed_name),
	           "external class name format used internally");
	
	    // Add class just loaded
	    // If a class loader supports parallel classloading handle parallel define requests
	    // find_or_define_instance_class may return a different instanceKlass
	    if (is_parallelCapable(class_loader)) {
	      k = find_or_define_instance_class(class_name, class_loader, k, THREAD);
	    } else {
	      define_instance_class(k, THREAD);
	    }
	  }
	
  {- -------------------------------------------
  (1) パース処理中に例外が出ていた場合は, 
      PlaceholderTable 中に残ってしまったゴミを片付け, 
      待っているスレッドを notify_all() で起こしてから, 
      NULL をリターン.
  
      (ただし, クラス名を見つける前にエラーになっていた場合 (parsed_name が NULL の場合) には,
      片付ける必要はないので, この処理はパスして次に進む)
      ---------------------------------------- -}

	  // If parsing the class file or define_instance_class failed, we
	  // need to remove the placeholder added on our behalf. But we
	  // must make sure parsed_name is valid first (it won't be if we had
	  // a format error before the class was parsed far enough to
	  // find the name).
	  if (HAS_PENDING_EXCEPTION && parsed_name != NULL) {
	    unsigned int p_hash = placeholders()->compute_hash(parsed_name,
	                                                       class_loader);
	    int p_index = placeholders()->hash_to_index(p_hash);
	    {
	    MutexLocker mu(SystemDictionary_lock, THREAD);
	    placeholders()->find_and_remove(p_index, p_hash, parsed_name, class_loader, THREAD);
	    SystemDictionary_lock->notify_all();
	    }
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (debug_only 時にのみ実行)
      ---------------------------------------- -}

	  // Make sure that we didn't leave a place holder in the
	  // SystemDictionary; this is only done on success
	  debug_only( {
	    if (!HAS_PENDING_EXCEPTION) {
	      assert(parsed_name != NULL, "parsed_name is still null?");
	      Symbol*  h_name    = k->name();
	      Handle h_loader (THREAD, k->class_loader());
	
	      MutexLocker mu(SystemDictionary_lock, THREAD);
	
	      klassOop check = find_class(parsed_name, class_loader);
	      assert(check == k(), "should be present in the dictionary");
	
	      klassOop check2 = find_class(h_name, h_loader);
	      assert(check == check2, "name inconsistancy in SystemDictionary");
	    }
	  } );
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return k();
	}
	
```


