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
void instanceRefKlass::release_and_notify_pending_list_lock(
  BasicLock *pending_list_basic_lock) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など) (See: PreserveExceptionMark)
      ---------------------------------------- -}

	  // we may enter this with pending exception set
	  PRESERVE_EXCEPTION_MARK;  // exceptions are never thrown, needed for TRAPS argument
	  //

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle h_lock(THREAD, java_lang_ref_Reference::pending_list_lock());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ObjectSynchronizer::current_thread_holds_lock(
	           JavaThread::current(), h_lock),
	         "Lock should be held");

  {- -------------------------------------------
  (1) java_lang_ref_Reference::pending_list() に対して待っているスレッドがいるかもしれないので, 
      ObjectSynchronizer::notifyall() を呼んで起床させておく.
      ---------------------------------------- -}

	  // Notify waiters on pending lists lock if there is any reference.
	  if (java_lang_ref_Reference::pending_list() != NULL) {
	    ObjectSynchronizer::notifyall(h_lock, THREAD);
	  }

  {- -------------------------------------------
  (1) java_lang_ref_Reference::pending_list_lock() に対して 
      ObjectSynchronizer::fast_exit() でロックを開放する.
      ---------------------------------------- -}

	  ObjectSynchronizer::fast_exit(h_lock(), pending_list_basic_lock, THREAD);

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (HAS_PENDING_EXCEPTION) CLEAR_PENDING_EXCEPTION;
	}
	
```


