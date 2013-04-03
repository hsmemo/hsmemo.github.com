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
  (1) ThreadStateTransition::transition() を呼び出すだけ.
      ---------------------------------------- -}

	   void trans(JavaThreadState from, JavaThreadState to)  { transition(_thread, from, to); }
	
```


