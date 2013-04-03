---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  void invariants() const { assert(top() >= start() && top() <= end(), "invalid tlab"); }
	
```


