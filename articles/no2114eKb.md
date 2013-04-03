---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp
### 説明(description)

```
// build a map of JavaThread to all its owned AbstractOwnableSynchronizer
```

### 名前(function name)
```
void ConcurrentLocksDump::build_map(GrowableArray<oop>* aos_objects) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (引数として java.util.concurrent.locks.AbstractOwnableSynchronizer オブジェクトの配列が渡される)
      この配列内を全て辿り, 
      java_util_concurrent_locks_AbstractOwnableSynchronizer::get_owner_threadObj() で
      ロックを確保されているかどうか調べていく.
      ロックを確保されている AbstractOwnableSynchronizer オブジェクトについては, 
      ConcurrentLocksDump::add_lock() で確保主のスレッドと紐づける.
      ---------------------------------------- -}

	  int length = aos_objects->length();
	  for (int i = 0; i < length; i++) {
	    oop o = aos_objects->at(i);
	    oop owner_thread_obj = java_util_concurrent_locks_AbstractOwnableSynchronizer::get_owner_threadObj(o);
	    if (owner_thread_obj != NULL) {
	      JavaThread* thread = java_lang_Thread::thread(owner_thread_obj);
	      assert(o->is_instance(), "Must be an instanceOop");
	      add_lock(thread, (instanceOop) o);
	    }
	  }
	}
	
```


