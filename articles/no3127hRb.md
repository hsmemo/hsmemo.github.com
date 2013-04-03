---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
  void do_object(oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CMTask::deal_with_reference() を呼び出すだけ.
      ---------------------------------------- -}

	    _task->deal_with_reference(obj);
	  }
	
```


