---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.cpp

### 名前(function name)
```
void PSYoungGen::compact() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) PSMarkSweepDecorator::compact() を3回呼び出し, 
      それぞれ Eden 領域, From 領域, To 領域内のオブジェクトに対して
      コンパクション処理を行う.
      ---------------------------------------- -}

	  eden_mark_sweep()->compact(ZapUnusedHeapArea);
	  from_mark_sweep()->compact(ZapUnusedHeapArea);
	  // Mark sweep stores preserved markOops in to space, don't disturb!
	  to_mark_sweep()->compact(false);
	}
	
```


