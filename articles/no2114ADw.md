---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.cpp

### 名前(function name)
```
void PSOldGen::adjust_pointers() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PSMarkSweepDecorator::adjust_pointers() を呼び出して, 
      New 領域内のオブジェクトに対して
      その中に含まれるポインタの修正を行う.
      ---------------------------------------- -}

	  object_mark_sweep()->adjust_pointers();
	}
	
```


