---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/os.cpp

### 名前(function name)
```
OSReturn os::set_priority(Thread* thread, ThreadPriority p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  if (!(!thread->is_Java_thread() ||
	         Thread::current() == thread  ||
	         Threads_lock->owned_by_self()
	         || thread->is_Compiler_thread()
	        )) {
	    assert(false, "possibility of dangling Thread pointer");
	  }
	#endif
	
  {- -------------------------------------------
  (1) java_to_os_priority 配列を用いて優先度の値をそれぞれの OS 用の値に換算した後, 
      os::set_native_priority() を呼んで実際に優先度を変える処理を行う.
  
      (ただし, 指定された優先度(p)が指定可能な値の範囲を超えている場合には, 
       何もせずに OS_ERR をリターンするだけ)
      ---------------------------------------- -}

	  if (p >= MinPriority && p <= MaxPriority) {
	    int priority = java_to_os_priority[p];
	    return set_native_priority(thread, priority);
	  } else {
	    assert(false, "Should not happen");
	    return OS_ERR;
	  }
	}
	
```


