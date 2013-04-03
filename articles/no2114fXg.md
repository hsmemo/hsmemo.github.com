---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Returns an array of java/lang/management/MemoryPoolMXBean object
// one for each memory pool if obj == null; otherwise returns
// an array of memory pools for a given memory manager if
// it is a valid memory manager.
```

### 名前(function name)
```
JVM_ENTRY(jobjectArray, jmm_GetMemoryPools(JNIEnv* env, jobject obj))
```

### 本体部(body)
```
	(この関数は, java/lang/management/MemoryPoolMXBean オブジェクトの配列を返す.
	 その際に, 引数で MemoryManager オブジェクトが指定されている場合は, 
	 その MemoryManager に関連した MemoryPool だけを返す.
	 逆に指定されていない場合 (= NULL の場合) には, 全ての MemoryPool を返す)
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	
  {- -------------------------------------------
  (1) まず, 引数で MemoryManager オブジェクトが指定されているかどうかを確認しておく.
  
      (なお, 指定されている(= NULL ではない)が正しい MemoryManager オブジェクトを指していない場合には, ここでリターン)
      ---------------------------------------- -}

	  int num_memory_pools;
	  MemoryManager* mgr = NULL;
	  if (obj == NULL) {
	    num_memory_pools = MemoryService::num_memory_pools();
	  } else {
	    mgr = get_memory_manager_from_jobject(obj, CHECK_NULL);
	    if (mgr == NULL) {
	      return NULL;
	    }
	    num_memory_pools = mgr->num_memory_pools();
	  }
	
  {- -------------------------------------------
  (1) 返値として返す MemoryPoolMXBean の配列を用意する (以下の poolArray).
      ---------------------------------------- -}

	  // Allocate the resulting MemoryPoolMXBean[] object
	  klassOop k = Management::java_lang_management_MemoryPoolMXBean_klass(CHECK_NULL);
	  instanceKlassHandle ik (THREAD, k);
	  objArrayOop r = oopFactory::new_objArray(ik(), num_memory_pools, CHECK_NULL);
	  objArrayHandle poolArray(THREAD, r);
	
  {- -------------------------------------------
  (1) mgrArray の中に, 実際の MemoryPool を表す 
      sun.management.MemoryPoolImpl オブジェクトを積めていく.
  
      引数で MemoryManager オブジェクトが指定されている場合には, 
      MemoryManager::get_memory_pool() で
      その MemoryManager に関連した MemoryPool だけを取得していく.
  
      そうでなければ, MemoryService::get_memory_pool() で
      全ての MemoryPool を取得していく.
    
      どちらの場合も, 配列に積める前には
      MemoryPool オブジェクトから sun.management.MemoryPoolImpl オブジェクトに変換するために
      MemoryPool::get_memory_pool_instance() が呼び出される.
      ---------------------------------------- -}

	  if (mgr == NULL) {
	    // Get all memory pools
	    for (int i = 0; i < num_memory_pools; i++) {
	      MemoryPool* pool = MemoryService::get_memory_pool(i);
	      instanceOop p = pool->get_memory_pool_instance(CHECK_NULL);
	      instanceHandle ph(THREAD, p);
	      poolArray->obj_at_put(i, ph());
	    }
	  } else {
	    // Get memory pools managed by a given memory manager
	    for (int i = 0; i < num_memory_pools; i++) {
	      MemoryPool* pool = mgr->get_memory_pool(i);
	      instanceOop p = pool->get_memory_pool_instance(CHECK_NULL);
	      instanceHandle ph(THREAD, p);
	      poolArray->obj_at_put(i, ph());
	    }
	  }

  {- -------------------------------------------
  (1) 結果を JNIHandle 化してリターン
      ---------------------------------------- -}

	  return (jobjectArray) JNIHandles::make_local(env, poolArray());
	JVM_END
	
```


