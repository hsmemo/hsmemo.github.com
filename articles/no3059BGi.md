---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp
### 説明(description)

```
// JNI functions only transform a pending async exception to a synchronous
// exception in ExceptionOccurred and ExceptionCheck calls, since
// delivering an async exception in other places won't change the native
// code's control flow and would be harmful when native code further calls
// JNI functions with a pending exception. Async exception is also checked
// during the call, so ExceptionOccurred/ExceptionCheck won't return
// false but deliver the async exception at the very end during
// state transition.

```

### 名前(function name)
```
static void jni_check_async_exceptions(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread == Thread::current(), "must be itself");

  {- -------------------------------------------
  (1) JavaThread::check_and_handle_async_exceptions() を呼び出して
      Asynchronous Exception のチェックをしておく.
      (もし Asynchronous Exception が存在していれば, この中で pending_exception にセットされる)
      ---------------------------------------- -}

	  thread->check_and_handle_async_exceptions();
	}
	
```


