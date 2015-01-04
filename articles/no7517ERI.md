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
klassOop SystemDictionary::resolve_instance_class_or_null(Symbol* name, Handle class_loader, Handle protection_domain, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(name != NULL && !FieldType::is_array(name) &&
	         !FieldType::is_obj(name), "invalid class name");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // UseNewReflection
	  // Fix for 4474172; see evaluation for more details
	  class_loader = Handle(THREAD, java_lang_ClassLoader::non_reflection_class_loader(class_loader()));
	
  {- -------------------------------------------
  (1) 既にロード済みでないかどうか調べる.
      もしロード済みであれば, ここでリターン.
      ---------------------------------------- -}

	  // Do lookup to see if class already exist and the protection domain
	  // has the right access
	  unsigned int d_hash = dictionary()->compute_hash(name, class_loader);
	  int d_index = dictionary()->hash_to_index(d_hash);
	  klassOop probe = dictionary()->find(d_index, d_hash, name, class_loader,
	                                      protection_domain, THREAD);
	  if (probe != NULL) return probe;
	
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (DoObjectLock は, ロックを取得するかどうかを示す. parallel に処理可能なクラスローダーなら取得しない)
      ---------------------------------------- -}

	  // Non-bootstrap class loaders will call out to class loader and
	  // define via jvm/jni_DefineClass which will acquire the
	  // class loader object lock to protect against multiple threads
	  // defining the class in parallel by accident.
	  // This lock must be acquired here so the waiter will find
	  // any successful result in the SystemDictionary and not attempt
	  // the define
	  // ParallelCapable Classloaders and the bootstrap classloader,
	  // or all classloaders with UnsyncloadClass do not acquire lock here
	  bool DoObjectLock = true;
	  if (is_parallelCapable(class_loader)) {
	    DoObjectLock = false;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  unsigned int p_hash = placeholders()->compute_hash(name, class_loader);
	  int p_index = placeholders()->hash_to_index(p_hash);
	
  {- -------------------------------------------
  (1) (以降の処理は ... で排他した状態で行う)
  
      なお, (プロファイル情報の記録) も行っている. ("sun.cls.systemLoaderLockContentionRate", "sun.cls.nonSystemLoaderLockContentionRate")
      (See: SystemDictionary::check_loader_lock_contention())
      ---------------------------------------- -}

	  // Class is not in SystemDictionary so we have to do loading.
	  // Make sure we are synchronized on the class loader before we proceed
	  Handle lockObject = compute_loader_lock_object(class_loader, THREAD);
	  check_loader_lock_contention(lockObject, THREAD);
	  ObjectLocker ol(lockObject, THREAD, DoObjectLock);
	
  {- -------------------------------------------
  (1) (コメントによると, 以降で (ロックを取った後で) もう一度ロード済みではないかどうかを確認する)
      ---------------------------------------- -}

	  // Check again (after locking) if class already exist in SystemDictionary

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  bool class_has_been_loaded   = false;
	  bool super_load_in_progress  = false;
	  bool havesupername = false;
	  instanceKlassHandle k;
	  PlaceholderEntry* placeholder;
	  Symbol* superclassname = NULL;
	
  {- -------------------------------------------
  (1) SystemDictionary::find_class() を呼んで, 既にロード済みでないかどうか調べる.
      (なお, この処理は SystemDictionary_lock を取得して排他した状態で行う)
      結果に応じて, 以下の変数を設定する.
  
      * class_has_been_loaded
        既にロード済みであれば, true
      * k
        既にロード済みであれば, 対象クラスの instanceKlassHandle
      * super_load_in_progress
        ロード済みではないが, ロード処理中(#TODO)であれば, true
      * superclassname
  
      * havesupername
  
      ---------------------------------------- -}

	  {
	    MutexLocker mu(SystemDictionary_lock, THREAD);
	    klassOop check = find_class(d_index, d_hash, name, class_loader);
	    if (check != NULL) {
	      // Klass is already loaded, so just return it
	      class_has_been_loaded = true;
	      k = instanceKlassHandle(THREAD, check);
	    } else {
	      placeholder = placeholders()->get_entry(p_index, p_hash, name, class_loader);
	      if (placeholder && placeholder->super_load_in_progress()) {
	         super_load_in_progress = true;
	         if (placeholder->havesupername() == true) {
	           superclassname = placeholder->supername();
	           havesupername = true;
	         }
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // If the class in is in the placeholder table, class loading is in progress
	  if (super_load_in_progress && havesupername==true) {
	    k = SystemDictionary::handle_parallel_super_load(name, superclassname,
	        class_loader, protection_domain, lockObject, THREAD);
	    if (HAS_PENDING_EXCEPTION) {
	      return NULL;
	    }
	    if (!k.is_null()) {
	      class_has_been_loaded = true;
	    }
	  }
	
  {- -------------------------------------------
  (1) まだクラスがロードされていなければ, 以下の if ブロック内でロード処理を行う.
      ---------------------------------------- -}

	  if (!class_has_been_loaded) {
	
    {- -------------------------------------------
  (1.1) コメントによると, 
        以降の処理で, ロード処理の責任者を決めるために PlaceholderEntry を追加する.
        これには 5つのケースがある.
        * 
        * 
        * 
        * 
        * 
        ---------------------------------------- -}

	    // add placeholder entry to record loading instance class
	    // Five cases:
	    // All cases need to prevent modifying bootclasssearchpath
	    // in parallel with a classload of same classname
	    // Redefineclasses uses existence of the placeholder for the duration
	    // of the class load to prevent concurrent redefinition of not completely
	    // defined classes.
	    // case 1. traditional classloaders that rely on the classloader object lock
	    //   - no other need for LOAD_INSTANCE
	    // case 2. traditional classloaders that break the classloader object lock
	    //    as a deadlock workaround. Detection of this case requires that
	    //    this check is done while holding the classloader object lock,
	    //    and that lock is still held when calling classloader's loadClass.
	    //    For these classloaders, we ensure that the first requestor
	    //    completes the load and other requestors wait for completion.
	    // case 3. UnsyncloadClass - don't use objectLocker
	    //    With this flag, we allow parallel classloading of a
	    //    class/classloader pair
	    // case4. Bootstrap classloader - don't own objectLocker
	    //    This classloader supports parallelism at the classloader level,
	    //    but only allows a single load of a class/classloader pair.
	    //    No performance benefit and no deadlock issues.
	    // case 5. parallelCapable user level classloaders - without objectLocker
	    //    Allow parallel classloading of a class/classloader pair

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    bool throw_circularity_error = false;

    {- -------------------------------------------
  (1.1) 以下のブロック内で PlaceholderEntry を PlaceholderTable 内に追加する.
        (なお, この処理は SystemDictionary_lock を取得して排他した状態で行う)
        ---------------------------------------- -}

	    {
	      MutexLocker mu(SystemDictionary_lock, THREAD);

      {- -------------------------------------------
  (1.1.1) まず, 対象クラスのロード処理が実行中かどうかを確認する.
          (ただし, ... の場合は並列にロード可能なので確認せず先に進む)
          
          ロード中だった場合は, 以下の処理を行う.
          * ロード処理を行っているスレッドがカレントスレッドだった場合: (= クラス階層が循環している場合) 
            throw_circularity_error 変数を true にするだけ.
          * 〃 カレントスレッドではない場合:
            他スレッドによるロード処理が完了するか, あるいは
            処理に失敗してロードしているスレッドがいなくなってしまうまで, 以下の処理を繰り返す.
            (1) ロード処理が終わるまで待機する
                (待機処理は, SystemDictionary_lock に対する Monitor::wait(), 又は SystemDictionary::double_lock_wait() で行う)
            (2) SystemDictionary::find_class() を呼んで, ロード済みかどうかを調べる.
                (ロード済みであれば, k 局所変数及び class_has_been_loaded 局所変数を設定)
            (3) 対象クラスのロード処理がまだ PlaceholderTable に登録されているかどうか調べる.
          ---------------------------------------- -}

	      if (class_loader.is_null() || !is_parallelCapable(class_loader)) {
	        PlaceholderEntry* oldprobe = placeholders()->get_entry(p_index, p_hash, name, class_loader);
	        if (oldprobe) {
	          // only need check_seen_thread once, not on each loop
	          // 6341374 java/lang/Instrument with -Xcomp
	          if (oldprobe->check_seen_thread(THREAD, PlaceholderTable::LOAD_INSTANCE)) {
	            throw_circularity_error = true;
	          } else {
	            // case 1: traditional: should never see load_in_progress.
	            while (!class_has_been_loaded && oldprobe && oldprobe->instance_load_in_progress()) {
	
	              // case 4: bootstrap classloader: prevent futile classloading,
	              // wait on first requestor
	              if (class_loader.is_null()) {
	                SystemDictionary_lock->wait();
	              } else {
	              // case 2: traditional with broken classloader lock. wait on first
	              // requestor.
	                double_lock_wait(lockObject, THREAD);
	              }
	              // Check if classloading completed while we were waiting
	              klassOop check = find_class(d_index, d_hash, name, class_loader);
	              if (check != NULL) {
	                // Klass is already loaded, so just return it
	                k = instanceKlassHandle(THREAD, check);
	                class_has_been_loaded = true;
	              }
	              // check if other thread failed to load and cleaned up
	              oldprobe = placeholders()->get_entry(p_index, p_hash, name, class_loader);
	            }
	          }
	        }
	      }

      {- -------------------------------------------
  (1.1.1) まだクラスがロードされていなければ (かつクラス階層の循環も検出されていなければ)
          PlaceholderTable::find_and_add() を呼んで
          ロード対象のクラスに対応する PlaceholderEntry を PlaceholderTable 内に登録する.
  
          なお, ObjectLocker(?) を使わないクラスローダーは並列に動いていた恐れがあるので, 
          PlaceholderEntry の登録後に, 念のために 
          SystemDictionary::find_class() を呼んで, ロード済みかどうかを調べている.
          もしロード済みであれば, (それをリターンするだけでいいので)
          登録した PlaceholderEntry を片付けて排他を解除し, 
          待っているスレッドを notify_all() で起床させておく.
          (そして, k 局所変数及び class_has_been_loaded 局所変数を設定)
          ---------------------------------------- -}

	      // All cases: add LOAD_INSTANCE
	      // case 3: UnsyncloadClass || case 5: parallelCapable: allow competing threads to try
	      // LOAD_INSTANCE in parallel
	      // add placeholder entry even if error - callers will remove on error
	      if (!throw_circularity_error && !class_has_been_loaded) {
	        PlaceholderEntry* newprobe = placeholders()->find_and_add(p_index, p_hash, name, class_loader, PlaceholderTable::LOAD_INSTANCE, NULL, THREAD);
	        // For class loaders that do not acquire the classloader object lock,
	        // if they did not catch another thread holding LOAD_INSTANCE,
	        // need a check analogous to the acquire ObjectLocker/find_class
	        // i.e. now that we hold the LOAD_INSTANCE token on loading this class/CL
	        // one final check if the load has already completed
	        // class loaders holding the ObjectLock shouldn't find the class here
	        klassOop check = find_class(d_index, d_hash, name, class_loader);
	        if (check != NULL) {
	        // Klass is already loaded, so just return it
	          k = instanceKlassHandle(THREAD, check);
	          class_has_been_loaded = true;
	          newprobe->remove_seen_thread(THREAD, PlaceholderTable::LOAD_INSTANCE);
	          placeholders()->find_and_remove(p_index, p_hash, name, class_loader, THREAD);
	          SystemDictionary_lock->notify_all();
	        }
	      }
	    }

    {- -------------------------------------------
  (1.1) もし, 以上の処理でクラス階層内に循環を検出した場合は, ClassCircularityError.
        ---------------------------------------- -}

	    // must throw error outside of owning lock
	    if (throw_circularity_error) {
	      ResourceMark rm(THREAD);
	      THROW_MSG_0(vmSymbols::java_lang_ClassCircularityError(), name->as_C_string());
	    }
	
    {- -------------------------------------------
  (1.1) まだクラスがロードされていなければ, 以下の if ブロック内でロード処理を行う.
        ---------------------------------------- -}

	    if (!class_has_been_loaded) {
	
      {- -------------------------------------------
  (1.1.1) SystemDictionary::load_instance_class() を呼んで, 実際のロード処理を行う.
          ---------------------------------------- -}

	      // Do actual loading
	      k = load_instance_class(name, class_loader, THREAD);
	
      {- -------------------------------------------
  (1.1.1) 
          ---------------------------------------- -}

	      // For UnsyncloadClass only
	      // If they got a linkageError, check if a parallel class load succeeded.
	      // If it did, then for bytecode resolution the specification requires
	      // that we return the same result we did for the other thread, i.e. the
	      // successfully loaded instanceKlass
	      // Should not get here for classloaders that support parallelism
	      // with the new cleaner mechanism, even with AllowParallelDefineClass
	      // Bootstrap goes through here to allow for an extra guarantee check
	      if (UnsyncloadClass || (class_loader.is_null())) {
	        if (k.is_null() && HAS_PENDING_EXCEPTION
	          && PENDING_EXCEPTION->is_a(SystemDictionary::LinkageError_klass())) {
	          MutexLocker mu(SystemDictionary_lock, THREAD);
	          klassOop check = find_class(d_index, d_hash, name, class_loader);
	          if (check != NULL) {
	            // Klass is already loaded, so just use it
	            k = instanceKlassHandle(THREAD, check);
	            CLEAR_PENDING_EXCEPTION;
	            guarantee((!class_loader.is_null()), "dup definition for bootstrap loader?");
	          }
	        }
	      }
	
      {- -------------------------------------------
  (1.1.1) 対応する PlaceholderEntry を片付けて排他を解除し, 
          待っているスレッドを notify_all() で起床させておく.
          (なお, この処理は SystemDictionary_lock を取得して排他した状態で行う)
          ---------------------------------------- -}

	      // clean up placeholder entries for success or error
	      // This cleans up LOAD_INSTANCE entries
	      // It also cleans up LOAD_SUPER entries on errors from
	      // calling load_instance_class
	      {
	        MutexLocker mu(SystemDictionary_lock, THREAD);
	        PlaceholderEntry* probe = placeholders()->get_entry(p_index, p_hash, name, class_loader);
	        if (probe != NULL) {
	          probe->remove_seen_thread(THREAD, PlaceholderTable::LOAD_INSTANCE);
	          placeholders()->find_and_remove(p_index, p_hash, name, class_loader, THREAD);
	          SystemDictionary_lock->notify_all();
	        }
	      }
	
      {- -------------------------------------------
  (1.1.1) もしロードが成功しているが (= 例外が出ておらず, クラスも取得できているが), 
          ...#TODO の場合, 
          SystemDictionary::check_constraints() を呼んで, 
          Java 仮想マシン仕様の「ロード制約(loading constraints)」をチェックする.
          (ロード制約に違反するようなクラスロードが起こると LinkageError)
  
          チェックで問題がなければ, SystemDictionary::update_dictionary() を呼んで
          実際に SystemDictionary に登録する.
          (なお, ここは (JVMTI のフック点) でもある (See: ClassLoad イベント))
          ---------------------------------------- -}

	      // If everything was OK (no exceptions, no null return value), and
	      // class_loader is NOT the defining loader, do a little more bookkeeping.
	      if (!HAS_PENDING_EXCEPTION && !k.is_null() &&
	        k->class_loader() != class_loader()) {
	
	        check_constraints(d_index, d_hash, k, class_loader, false, THREAD);
	
	        // Need to check for a PENDING_EXCEPTION again; check_constraints
	        // can throw and doesn't use the CHECK macro.
	        if (!HAS_PENDING_EXCEPTION) {
	          { // Grabbing the Compile_lock prevents systemDictionary updates
	            // during compilations.
	            MutexLocker mu(Compile_lock, THREAD);
	            update_dictionary(d_index, d_hash, p_index, p_hash,
	                            k, class_loader, THREAD);
	          }
	          if (JvmtiExport::should_post_class_load()) {
	            Thread *thread = THREAD;
	            assert(thread->is_Java_thread(), "thread->is_Java_thread()");
	            JvmtiExport::post_class_load((JavaThread *) thread, k());
	          }
	        }
	      }

      {- -------------------------------------------
  (1.1.1) もしロードが失敗していた場合は (例外が出ている場合 or クラスが取得できていない場合), 
          PlaceholderTable 中に残ってしまったゴミを片付け, 
          待っているスレッドを notify_all() で起こしてから, 
          NULL をリターン.
          ---------------------------------------- -}

	      if (HAS_PENDING_EXCEPTION || k.is_null()) {
	        // On error, clean up placeholders
	        {
	          MutexLocker mu(SystemDictionary_lock, THREAD);
	          placeholders()->find_and_remove(p_index, p_hash, name, class_loader, THREAD);
	          SystemDictionary_lock->notify_all();
	        }
	        return NULL;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  {
	    Handle loader (THREAD, k->class_loader());
	    MutexLocker mu(SystemDictionary_lock, THREAD);
	    oop kk = find_class(name, loader);
	    assert(kk == k(), "should be present in dictionary");
	  }
	#endif
	
  {- -------------------------------------------
  (1) protection domain によるチェックを行う. 
      問題がある場合には例外が起こる. 問題がなければ結果をリターン.
  
      * protection domain が指定されていない場合 (= protection_domain 引数が null Handle の場合)
        チェックの必要はないので, そのまま結果をリターン.
  
      * 〃指定されている場合
        (1) Dictionary::is_valid_protection_domain() でチェックを行い, 
            問題なければ (true であれば), ここで結果をリターン.
            (なお, この処理は SystemDictionary_lock を取得して排他した状態で行う)
        (2) 次は, SystemDictionary::validate_protection_domain() でチェックを行い, 
            問題なければ (例外が出なければ), 結果をリターン.
      ---------------------------------------- -}

	  // return if the protection domain in NULL
	  if (protection_domain() == NULL) return k();
	
	  // Check the protection domain has the right access
	  {
	    MutexLocker mu(SystemDictionary_lock, THREAD);
	    // Note that we have an entry, and entries can be deleted only during GC,
	    // so we cannot allow GC to occur while we're holding this entry.
	    // We're using a No_Safepoint_Verifier to catch any place where we
	    // might potentially do a GC at all.
	    // SystemDictionary::do_unloading() asserts that classes are only
	    // unloaded at a safepoint.
	    No_Safepoint_Verifier nosafepoint;
	    if (dictionary()->is_valid_protection_domain(d_index, d_hash, name,
	                                                 class_loader,
	                                                 protection_domain)) {
	      return k();
	    }
	  }
	
	  // Verify protection domain. If it fails an exception is thrown
	  validate_protection_domain(k, class_loader, protection_domain, CHECK_(klassOop(NULL)));
	
	  return k();
	}
	
```


