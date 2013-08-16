---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp
### 説明(description)

```
// Returns an array of all live Thread objects (VM internal JavaThreads,
// jvmti agent threads, and JNI attaching threads  are skipped)
// See CR 6404306 regarding JNI attaching threads
```

### 名前(function name)
```
JVM_ENTRY(jobjectArray, JVM_GetAllThreads(JNIEnv *env, jclass dummy))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);

  {- -------------------------------------------
  (1) ThreadsListEnumerator のコンストラクタで, 全スレッドの一覧を取得.
      ---------------------------------------- -}

	  ThreadsListEnumerator tle(THREAD, false, false);

  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: JvmtiVMObjectAllocEventCollector)
      ---------------------------------------- -}

	  JvmtiVMObjectAllocEventCollector oam;
	
  {- -------------------------------------------
  (1) 返値は java.lang.Thread[] として返す必要があるため, 
      結果を入れる java.lang.Thread 配列を確保.
      ---------------------------------------- -}

	  int num_threads = tle.num_threads();
	  objArrayOop r = oopFactory::new_objArray(SystemDictionary::Thread_klass(), num_threads, CHECK_NULL);
	  objArrayHandle threads_ah(THREAD, r);
	
  {- -------------------------------------------
  (1) ThreadsListEnumerator::get_threadObj() を使って
      各スレッドに対応する java.lang.Thread オブジェクトを取得し, 
      結果の配列に格納していく.
      ---------------------------------------- -}

	  for (int i = 0; i < num_threads; i++) {
	    Handle h = tle.get_threadObj(i);
	    threads_ah->obj_at_put(i, h());
	  }
	
  {- -------------------------------------------
  (1) 結果を JNIHandle 化してリターン.
      ---------------------------------------- -}

	  return (jobjectArray) JNIHandles::make_local(env, threads_ah());
	JVM_END
	
```


