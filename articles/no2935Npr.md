---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _interp_only_mode フィールドの値をデクリメントするだけ.
      ---------------------------------------- -}

	  void decrement_interp_only_mode()         { --_interp_only_mode; }
	
```


