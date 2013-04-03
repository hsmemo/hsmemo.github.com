---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)
(この関数は, java/lang/management/MemoryManagerMXBean オブジェクトの配列を返す.
 その際に, 引数で MemoryPool オブジェクトが指定されている場合は, 
 その MemoryPool に関連した MemoryManager だけを返す.
 逆に指定されていない場合 (= NULL の場合) には, 全ての MemoryManager を返す)

```
// Returns an array of java/lang/management/MemoryManagerMXBean object
// one for each memory manager if obj == null; otherwise returns
// an array of memory managers for a given memory pool if
// it is a valid memory pool.
```

### 名前(function name)
```
JVM_ENTRY(jobjectArray, jmm_GetMemoryManagers(JNIEnv* env, jobject obj))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	
  {- -------------------------------------------
  (1) まず, 引数で MemoryPool オブジェクトが指定されているかどうかを確認しておく.
  
      (なお, 指定されている(= NULL ではない)が正しい MemoryPool オブジェクトを指していない場合には, ここでリターン)
      ---------------------------------------- -}

	  int num_mgrs;
	  MemoryPool* pool = NULL;
	  if (obj == NULL) {
	    num_mgrs = MemoryService::num_memory_managers();
	  } else {
	    pool = get_memory_pool_from_jobject(obj, CHECK_NULL);
	    if (pool == NULL) {
	      return NULL;
	    }
	    num_mgrs = pool->num_memory_managers();
	  }
	
  {- -------------------------------------------
  (1) 返値として返す MemoryManagerMXBean の配列を用意する (以下の mgrArray).
      ---------------------------------------- -}

	  // Allocate the resulting MemoryManagerMXBean[] object
	  klassOop k = Management::java_lang_management_MemoryManagerMXBean_klass(CHECK_NULL);
	  instanceKlassHandle ik (THREAD, k);
	  objArrayOop r = oopFactory::new_objArray(ik(), num_mgrs, CHECK_NULL);
	  objArrayHandle mgrArray(THREAD, r);
	
  {- -------------------------------------------
  (1) mgrArray の中に, 実際の MemoryManager を表す sun.management.MemoryManagerImpl 
      (もしくは sun.management.GarbageCollectorImpl) オブジェクトを積めていく.
  
      引数で MemoryPool オブジェクトが指定されている場合には, 
      MemoryPool::get_memory_manager() で
      その MemoryPool に関連した MemoryManager だけを取得していく.
  
      そうでなければ, MemoryService::get_memory_manager() で
      全ての MemoryManager を取得していく.
    
      どちらの場合も, 配列に積める前には
      MemoryManager オブジェクトから sun.management.MemoryManagerImpl 
      (もしくは sun.management.GarbageCollectorImpl) オブジェクトに変換するために
      MemoryManager::get_memory_manager_instance() が呼び出される.
      ---------------------------------------- -}

	  if (pool == NULL) {
	    // Get all memory managers
	    for (int i = 0; i < num_mgrs; i++) {
	      MemoryManager* mgr = MemoryService::get_memory_manager(i);
	      instanceOop p = mgr->get_memory_manager_instance(CHECK_NULL);
	      instanceHandle ph(THREAD, p);
	      mgrArray->obj_at_put(i, ph());
	    }
	  } else {
	    // Get memory managers for a given memory pool
	    for (int i = 0; i < num_mgrs; i++) {
	      MemoryManager* mgr = pool->get_memory_manager(i);
	      instanceOop p = mgr->get_memory_manager_instance(CHECK_NULL);
	      instanceHandle ph(THREAD, p);
	      mgrArray->obj_at_put(i, ph());
	    }
	  }

  {- -------------------------------------------
  (1) 結果を JNIHandle 化してリターン
      ---------------------------------------- -}

	  return (jobjectArray) JNIHandles::make_local(env, mgrArray());
	JVM_END
	
```


