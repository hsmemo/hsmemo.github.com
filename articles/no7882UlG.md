---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
  ~ThreadBlockInVM() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadStateTransition::trans_and_fence() を呼んで, JavaThreadState を _thread_in_vm に変更する.
      ---------------------------------------- -}

	    trans_and_fence(_thread_blocked, _thread_in_vm);
	    // We don't need to clear_walkable because it will happen automagically when we return to java
	  }
	
```


