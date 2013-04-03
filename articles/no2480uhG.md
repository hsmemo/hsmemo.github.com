---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.inline.hpp

### 名前(function name)
```
inline void ParallelScavengeHeap::invoke_full_gc(bool maximum_compaction)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) UseParallelOldGC オプションに応じて, 以下のどちらかを呼び出す.
      * PSParallelCompact::invoke()
      * PSMarkSweep::invoke()
      ---------------------------------------- -}

	  if (UseParallelOldGC) {
	    PSParallelCompact::invoke(maximum_compaction);
	  } else {
	    PSMarkSweep::invoke(maximum_compaction);
	  }
	}
	
```


