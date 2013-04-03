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
  (1) ThreadStateTransition::transition_from_java() を呼び出すだけ.
      ---------------------------------------- -}

	   void trans_from_java(JavaThreadState to)              { transition_from_java(_thread, to); }
	
```


