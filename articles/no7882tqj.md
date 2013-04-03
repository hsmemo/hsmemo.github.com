---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadStateTransition::transition_and_fence() を呼び出すだけ.
      ---------------------------------------- -}

	   void trans_and_fence(JavaThreadState from, JavaThreadState to) { transition_and_fence(_thread, from, to); }
	
```


