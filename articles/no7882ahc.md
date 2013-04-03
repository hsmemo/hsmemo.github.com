---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceRefKlass.cpp

### 名前(function name)
```
void instanceRefKlass::acquire_pending_list_lock(BasicLock *pending_list_basic_lock) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など) (See: PreserveExceptionMark)
      ---------------------------------------- -}

	  // we may enter this with pending exception set
	  PRESERVE_EXCEPTION_MARK;  // exceptions are never thrown, needed for TRAPS argument

  {- -------------------------------------------
  (1) java_lang_ref_Reference::pending_list_lock() に対して 
      ObjectSynchronizer::fast_enter() でロックを取得する.
      ---------------------------------------- -}

	  Handle h_lock(THREAD, java_lang_ref_Reference::pending_list_lock());
	  ObjectSynchronizer::fast_enter(h_lock, pending_list_basic_lock, false, THREAD);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ObjectSynchronizer::current_thread_holds_lock(
	           JavaThread::current(), h_lock),
	         "Locking should have succeeded");

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (HAS_PENDING_EXCEPTION) CLEAR_PENDING_EXCEPTION;
	}
	
```


