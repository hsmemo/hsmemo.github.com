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
void PSOldGen::compact() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PSMarkSweepDecorator::compact() を呼び出すだけ.
      ---------------------------------------- -}

	  object_mark_sweep()->compact(ZapUnusedHeapArea);
	}
	
```


