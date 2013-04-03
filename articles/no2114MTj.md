---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp

### 名前(function name)
```
void PSMarkSweep::deallocate_stacks() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の Stack オブジェクトをクリアする.
      ---------------------------------------- -}

	  _preserved_mark_stack.clear(true);
	  _preserved_oop_stack.clear(true);
	  _marking_stack.clear();
	  _objarray_stack.clear(true);
	  _revisit_klass_stack.clear(true);
	  _revisit_mdo_stack.clear(true);
	}
	
```


