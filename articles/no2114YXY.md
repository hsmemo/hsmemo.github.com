---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vm_operations.cpp

### 名前(function name)
```
void VM_ThreadStop::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "must be at a safepoint");

  {- -------------------------------------------
  (1) JavaThread::send_thread_stop() を呼んで, 
      処理対象のスレッド内で例外を発生させる.
      
      (なお, 処理対象のスレッドが見つからない(= まだ開始していない or 既に死んでいる)場合には, 何もしない)
      ---------------------------------------- -}

	  JavaThread* target = java_lang_Thread::thread(target_thread());
	  // Note that this now allows multiple ThreadDeath exceptions to be
	  // thrown at a thread.
	  if (target != NULL) {
	    // the thread has run and is not already in the process of exiting
	    target->send_thread_stop(throwable());
	  }
	}
	
```


