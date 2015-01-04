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
// Support parallel classloading
// All parallel class loaders, including bootstrap classloader
// lock a placeholder entry for this class/class_loader pair
// to allow parallel defines of different classes for this class loader
// With AllowParallelDefine flag==true, in case they do not synchronize around
// FindLoadedClass/DefineClass, calls, we check for parallel
// loading for them, wait if a defineClass is in progress
// and return the initial requestor's results
// This flag does not apply to the bootstrap classloader.
// With AllowParallelDefine flag==false, call through to define_instance_class
// which will throw LinkageError: duplicate class definition.
// False is the requested default.
// For better performance, the class loaders should synchronize
// findClass(), i.e. FindLoadedClass/DefineClassIfAbsent or they
// potentially waste time reading and parsing the bytestream.
// Note: VM callers should ensure consistency of k/class_name,class_loader
```

### 名前(function name)
```
instanceKlassHandle SystemDictionary::find_or_define_instance_class(Symbol* class_name, Handle class_loader, instanceKlassHandle k, TRAPS) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  instanceKlassHandle nh = instanceKlassHandle(); // null Handle
	  Symbol*  name_h = k->name(); // passed in class_name may be null
	
	  unsigned int d_hash = dictionary()->compute_hash(name_h, class_loader);
	  int d_index = dictionary()->hash_to_index(d_hash);
	
	// Hold SD lock around find_class and placeholder creation for DEFINE_CLASS
	  unsigned int p_hash = placeholders()->compute_hash(name_h, class_loader);
	  int p_index = placeholders()->hash_to_index(p_hash);
	  PlaceholderEntry* probe;
	
  {- -------------------------------------------
  (1) 処理対象のクラスが
      「既にロードされているかどうか」および「現在他のスレッドがロード中かどうか」を調べる.
      * 既にロード済みであれば, ここでリターン.
      * 現在他のスレッドがロード中であれば, それが終わるまで以下のブロック内で待機.
        (待機処理は, SystemDictionary_lock に対して Monitor::wait() を呼ぶことで行う)
      * 上記のどちらでもなければ, 対象のクラスの処理スレッドとして自分自身を登録.
  
      (なお, この処理は SystemDictionary_lock を取得して排他した状態で行う)
  
      (See: Placeholder)
      ---------------------------------------- -}

	  {
	    MutexLocker mu(SystemDictionary_lock, THREAD);

    {- -------------------------------------------
  (1.1) 既にロード済みであれば, ここでリターン.
        ---------------------------------------- -}

	    // First check if class already defined
	    if (UnsyncloadClass || (is_parallelDefine(class_loader))) {
	      klassOop check = find_class(d_index, d_hash, name_h, class_loader);
	      if (check != NULL) {
	        return(instanceKlassHandle(THREAD, check));
	      }
	    }
	
    {- -------------------------------------------
  (1.1) 処理対象のクラスに関する PlaceholderEntry を取得.
        ---------------------------------------- -}

	    // Acquire define token for this class/classloader
	    probe = placeholders()->find_and_add(p_index, p_hash, name_h, class_loader, PlaceholderTable::DEFINE_CLASS, NULL, THREAD);

    {- -------------------------------------------
  (1.1) 現在他のスレッドがロード中であれば, それが終わるまで以下のブロック内で待機し, リターン.
        (待機処理は, SystemDictionary_lock に対して Monitor::wait() を呼ぶことで行う)
        ---------------------------------------- -}

	    // Wait if another thread defining in parallel
	    // All threads wait - even those that will throw duplicate class: otherwise
	    // caller is surprised by LinkageError: duplicate, but findLoadedClass fails
	    // if other thread has not finished updating dictionary
	    while (probe->definer() != NULL) {
	      SystemDictionary_lock->wait();
	    }
	    // Only special cases allow parallel defines and can use other thread's results
	    // Other cases fall through, and may run into duplicate defines
	    // caught by finding an entry in the SystemDictionary
	    if ((UnsyncloadClass || is_parallelDefine(class_loader)) && (probe->instanceKlass() != NULL)) {
	        probe->remove_seen_thread(THREAD, PlaceholderTable::DEFINE_CLASS);
	        placeholders()->find_and_remove(p_index, p_hash, name_h, class_loader, THREAD);
	        SystemDictionary_lock->notify_all();
	#ifdef ASSERT
	        klassOop check = find_class(d_index, d_hash, name_h, class_loader);
	        assert(check != NULL, "definer missed recording success");
	#endif
	        return(instanceKlassHandle(THREAD, probe->instanceKlass()));

    {- -------------------------------------------
  (1.1) 上記のどちらでもなければ, 対象のクラスの処理スレッドとして自分自身を登録.
        ---------------------------------------- -}

	    } else {
	      // This thread will define the class (even if earlier thread tried and had an error)
	      probe->set_definer(THREAD);
	    }
	  }
	
  {- -------------------------------------------
  (1) SystemDictionary::define_instance_class() を呼んで, SystemDictionary への登録を行う.
      ---------------------------------------- -}

	  define_instance_class(k, THREAD);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle linkage_exception = Handle(); // null handle
	
  {- -------------------------------------------
  (1) 対応する PlaceholderEntry を片付けて排他を解除し, 
      待っているスレッドを notify_all() で起床させておく.
  
      (なお, 例外が発生していた場合は, 
       (ロックを握っている間は投げられないので)
       ここでは linkage_exception 局所変数に入れるだけにし, 後で投げる)
  
      (ところで, assert(probe != NULL) があるのに, if でのチェックもあるのは...?? #TODO)
      ---------------------------------------- -}

	  // definer must notify any waiting threads
	  {
	    MutexLocker mu(SystemDictionary_lock, THREAD);
	    PlaceholderEntry* probe = placeholders()->get_entry(p_index, p_hash, name_h, class_loader);
	    assert(probe != NULL, "DEFINE_CLASS placeholder lost?");
	    if (probe != NULL) {
	      if (HAS_PENDING_EXCEPTION) {
	        linkage_exception = Handle(THREAD,PENDING_EXCEPTION);
	        CLEAR_PENDING_EXCEPTION;
	      } else {
	        probe->set_instanceKlass(k());
	      }
	      probe->set_definer(NULL);
	      probe->remove_seen_thread(THREAD, PlaceholderTable::DEFINE_CLASS);
	      placeholders()->find_and_remove(p_index, p_hash, name_h, class_loader, THREAD);
	      SystemDictionary_lock->notify_all();
	    }
	  }
	
  {- -------------------------------------------
  (1) 例外が発生していた場合は, ここで投げ直す.
      ---------------------------------------- -}

	  // Can't throw exception while holding lock due to rank ordering
	  if (linkage_exception() != NULL) {
	    THROW_OOP_(linkage_exception(), nh); // throws exception and returns
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return k;
	}
	
```


