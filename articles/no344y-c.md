---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/generationSizer.hpp

### 名前(function name)
```
  void initialize_flags() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) TwoGenerationCollectorPolicy::initialize_flags() を呼んで, 
      ヒープサイズに関する各種コマンドラインオプション(特に 2世代の世代別 GC に関するもの)の値を調整する.
      ---------------------------------------- -}

	    // Do basic sizing work
	    this->TwoGenerationCollectorPolicy::initialize_flags();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    // If the user hasn't explicitly set the number of worker
	    // threads, set the count.
	    assert(UseSerialGC ||
	           !FLAG_IS_DEFAULT(ParallelGCThreads) ||
	           (ParallelGCThreads > 0),
	           "ParallelGCThreads should be set before flag initialization");
	
  {- -------------------------------------------
  (1) MinSurvivorRatio と InitialSurvivorRatio を調整する.
      (どちらも, 値が 3 未満であれば 3 に切り上げる)
      ---------------------------------------- -}

	    // The survivor ratio's are calculated "raw", unlike the
	    // default gc, which adds 2 to the ratio value. We need to
	    // make sure the values are valid before using them.
	    if (MinSurvivorRatio < 3) {
	      MinSurvivorRatio = 3;
	    }
	
	    if (InitialSurvivorRatio < 3) {
	      InitialSurvivorRatio = 3;
	    }
	  }
	
```


