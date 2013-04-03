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
  (1) (単なる getter method)
      ---------------------------------------- -}

	  // Thread oop. threadObj() can be NULL for initial JavaThread
	  // (or for threads attached via JNI)
	  oop threadObj() const                          { return _threadObj; }
	
```


