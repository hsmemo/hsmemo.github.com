---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psCompactionManager.cpp

### 名前(function name)
```
void ParCompactionManager::drain_region_stacks() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の do...while ブロック内で, 
       ParCompactionManager が使用するタスクキュー(region_stack())内の region の処理を行う.
       この処理は, タスクキューが空になるまで繰り返される.)
      ---------------------------------------- -}

	  do {

  {- -------------------------------------------
  (1) まず, タスクキューの overflow stack 内の全 region を PSParallelCompact::fill_and_update_region() で処理する.
      ---------------------------------------- -}

	    // Drain overflow stack first so other threads can steal.
	    size_t region_index;
	    while (region_stack()->pop_overflow(region_index)) {
	      PSParallelCompact::fill_and_update_region(this, region_index);
	    }
	
  {- -------------------------------------------
  (1) 次に, タスクキュー内の region を PSParallelCompact::fill_and_update_region() で処理する.
      ---------------------------------------- -}

	    while (region_stack()->pop_local(region_index)) {
	      PSParallelCompact::fill_and_update_region(this, region_index);
	    }

  {- -------------------------------------------
  (1) タスクキューが空になるまで繰り返し.
      ---------------------------------------- -}

	  } while (!region_stack()->is_empty());
	}
	
```


