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
  ~ThreadInVMfromUnknown()  {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もしコンストラクタで実際に状態変更処理を行っていれば (= _thread フィールドが空でなければ), 
      ThreadStateTransition::transition_and_fence() を呼んで, JavaThreadState を _thread_in_native に戻す.
      ---------------------------------------- -}

	    if (_thread) {
	      ThreadStateTransition::transition_and_fence(_thread, _thread_in_vm, _thread_in_native);
	    }
	  }
	
```


