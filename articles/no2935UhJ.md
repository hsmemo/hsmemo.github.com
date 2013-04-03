---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
bool JvmtiDeferredEventQueue::has_events() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Service_lock->owned_by_self(), "Must own Service_lock");

  {- -------------------------------------------
  (1) 中身が入っていれば true をリターン.
      ---------------------------------------- -}

	  return _queue_head != NULL || _pending_list != NULL;
	}
	
```


