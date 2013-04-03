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
  ~ThreadInVMfromNative() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadStateTransition::trans_and_fence() を呼んで, JavaThreadState を _thread_in_native に変更する.
      ---------------------------------------- -}

	    trans_and_fence(_thread_in_vm, _thread_in_native);
	  }
	
```


