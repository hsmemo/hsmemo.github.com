---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp

### 名前(function name)
```
  void restore() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopDesc::set_mark() で mark 値の復帰を行う.
      ---------------------------------------- -}

	    _obj->set_mark(_mark);
	  }
	
```


