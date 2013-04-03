---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
void ConcurrentLocksDump::dump_at_safepoint() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では, 現在ロックされているシンクロナイザの一覧を取得する)
      ---------------------------------------- -}

	  // dump all locked concurrent locks

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "all threads are stopped");
	
  {- -------------------------------------------
  (1) (JDK のバージョンが 1.6 未満の場合には, 何もすることはない)
      ---------------------------------------- -}

	  if (JDK_Version::is_gte_jdk16x_version()) {

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    ResourceMark rm;
	
  {- -------------------------------------------
  (1) HeapInspection::find_instances_at_safepoint() で, 
      ヒープ中に存在する全ての java.util.concurrent.locks.AbstractOwnableSynchronizer オブジェクト
      (やそのサブクラスのオブジェクト)を取得する.
      ---------------------------------------- -}

	    GrowableArray<oop>* aos_objects = new GrowableArray<oop>(INITIAL_ARRAY_SIZE);
	
	    // Find all instances of AbstractOwnableSynchronizer
	    HeapInspection::find_instances_at_safepoint(SystemDictionary::abstract_ownable_synchronizer_klass(),
	                                                aos_objects);

  {- -------------------------------------------
  (1) ConcurrentLocksDump::build_map() で, 
      取得した AbstractOwnableSynchronizer オブジェクトの情報を
      そのオブジェクトのロックを確保しているスレッドに紐づける.
      ---------------------------------------- -}

	    // Build a map of thread to its owned AQS locks
	    build_map(aos_objects);
	  }
	}
	
```


