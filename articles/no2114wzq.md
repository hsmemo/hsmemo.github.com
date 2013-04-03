---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryPool.cpp
### 説明(description)

```
// Returns an instanceHandle of a MemoryPool object.
// It creates a MemoryPool instance when the first time
// this function is called.
```

### 名前(function name)
```
instanceOop MemoryPool::get_memory_pool_instance(TRAPS) {
```

### 本体部(body)
```
	(この関数では, 最初に呼ばれた際に MemoryPool インスタンスを生成し, 
	 二回目以降は最初に作ったインスタンスを流用する.)
  {- -------------------------------------------
  (1) 並行して, 他のスレッドがインスタンスを作り 
      _memory_pool_obj フィールドの書き換えを行っているかもしれないので, 
      _memory_pool_obj フィールドの読み取りは OrderAccess::load_ptr_acquire() で行う.
      (_memory_pool_obj フィールドのポインタだけは見えたが, 
       ポインタの指示先の同期は取られてなかった, という事態を防ぐ)
      ---------------------------------------- -}

	  // Must do an acquire so as to force ordering of subsequent
	  // loads from anything _memory_pool_obj points to or implies.
	  instanceOop pool_obj = (instanceOop)OrderAccess::load_ptr_acquire(&_memory_pool_obj);

  {- -------------------------------------------
  (1) _memory_pool_obj フィールドがまだ NULL の場合のみ, 以下の if 文中で新しいインスタンスを生成する.
      (一度インスタンスを作って _memory_pool_obj フィールドにキャッシュ済みの場合には, 新しく作ることはしない)
      ---------------------------------------- -}

	  if (pool_obj == NULL) {

  {- -------------------------------------------
  (1) sun.management.ManagementFactory.createMemoryPool() を呼び出して 
      新しい sun.management.MemoryPoolImpl オブジェクトを生成する.
  
      (なお, このインスタンス生成処理はロックで保護していないが, 複数のスレッドが実行してしまっても問題ない.
       余分に作ってしまっても GC で消えるだけなので)
      ---------------------------------------- -}

	    // It's ok for more than one thread to execute the code up to the locked region.
	    // Extra pool instances will just be gc'ed.
	    klassOop k = Management::sun_management_ManagementFactory_klass(CHECK_NULL);
	    instanceKlassHandle ik(THREAD, k);
	
	    Handle pool_name = java_lang_String::create_from_str(_name, CHECK_NULL);
	    jlong usage_threshold_value = (_usage_threshold->is_high_threshold_supported() ? 0 : -1L);
	    jlong gc_usage_threshold_value = (_gc_usage_threshold->is_high_threshold_supported() ? 0 : -1L);
	
	    JavaValue result(T_OBJECT);
	    JavaCallArguments args;
	    args.push_oop(pool_name);           // Argument 1
	    args.push_int((int) is_heap());     // Argument 2
	
	    Symbol* method_name = vmSymbols::createMemoryPool_name();
	    Symbol* signature = vmSymbols::createMemoryPool_signature();
	
	    args.push_long(usage_threshold_value);    // Argument 3
	    args.push_long(gc_usage_threshold_value); // Argument 4
	
	    JavaCalls::call_static(&result,
	                           ik,
	                           method_name,
	                           signature,
	                           &args,
	                           CHECK_NULL);
	
	    instanceOop p = (instanceOop) result.get_jobject();
	    instanceHandle pool(THREAD, p);
	
  {- -------------------------------------------
  (1) _memory_pool_obj フィールドに作ったオブジェクトを格納する.
      この処理は Management_lock のロックを取って排他的に行う.
    
      なお, ここまでの処理の間に, 他のスレッドが _memory_pool_obj フィールドを書き換えているかもしれないので, 
      ロックの取得後に, もう一度 _memory_pool_obj フィールドを確認する.
      (Management_lock のロック処理で acquire を取っているので, 
       確認のためのロード処理がこれより上にリオーダされることはないが,  
       確認結果が NULL でなければそれを返値としてリターンするので
       確認自体もやっぱり OrderAccess::load_ptr_acquire() で行う.
       (返値を受け取った側で, ポインタだけは見えたが指示先の同期は取られてなかった, ということになりかねないので))
  
      実際の _memory_pool_obj フィールドの書き換えは, 
      (上の OrderAccess::load_ptr_acquire() で見えるように) OrderAccess::release_store_ptr() で行う.
      ---------------------------------------- -}

	    {
	      // Get lock since another thread may have create the instance
	      MutexLocker ml(Management_lock);
	
	      // Check if another thread has created the pool.  We reload
	      // _memory_pool_obj here because some other thread may have
	      // initialized it while we were executing the code before the lock.
	      //
	      // The lock has done an acquire, so the load can't float above it,
	      // but we need to do a load_acquire as above.
	      pool_obj = (instanceOop)OrderAccess::load_ptr_acquire(&_memory_pool_obj);
	      if (pool_obj != NULL) {
	         return pool_obj;
	      }
	
	      // Get the address of the object we created via call_special.
	      pool_obj = pool();
	
	      // Use store barrier to make sure the memory accesses associated
	      // with creating the pool are visible before publishing its address.
	      // The unlock will publish the store to _memory_pool_obj because
	      // it does a release first.
	      OrderAccess::release_store_ptr(&_memory_pool_obj, pool_obj);
	    }
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return pool_obj;
	}
	
```


