---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) (単なる setter method)
      ---------------------------------------- -}

	  void set_in_progress()   { assert(_started, "must be starting a cycle"); _in_progress = true;  }
	
```


