---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp
### 説明(description)

```
  // Same as above, but assumes from = _thread_in_Java. This is simpler, since we
  // never block on entry to the VM. This will break the code, since e.g. preserve arguments
  // have not been setup.
```

### 名前(function name)
```
  static inline void transition_from_java(JavaThread *thread, JavaThreadState to) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(thread->thread_state() == _thread_in_Java, "coming from wrong thread state");

  {- -------------------------------------------
  (1) JavaThread::set_thread_state() を呼んで JavaThreadState を変更する.
      ---------------------------------------- -}

	    thread->set_thread_state(to);
	  }
	
```


