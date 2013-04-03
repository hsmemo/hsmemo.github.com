---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/collectorPolicy.cpp

### 名前(function name)
```
void TwoGenerationCollectorPolicy::initialize_flags() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenCollectorPolicy::initialize_flags() を呼んで, 
      ヒープサイズに関する各種コマンドラインオプション(特に世代別 GC に関するもの)の値を調整する.
      ---------------------------------------- -}

	  GenCollectorPolicy::initialize_flags();
	
  {- -------------------------------------------
  (1) ヒープサイズに関する各種コマンドラインオプション(特に 2世代の世代別 GC に関するもの)の値を調整する.
      ---------------------------------------- -}

	  OldSize = align_size_down(OldSize, min_alignment());
	  if (NewSize + OldSize > MaxHeapSize) {
	    MaxHeapSize = NewSize + OldSize;
	  }
	  MaxHeapSize = align_size_up(MaxHeapSize, max_alignment());
	
  {- -------------------------------------------
  (1) always_do_update_barrier 大域変数の値を初期化する.
      ---------------------------------------- -}

	  always_do_update_barrier = UseConcMarkSweepGC;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Check validity of heap flags
	  assert(OldSize     % min_alignment() == 0, "old space alignment");
	  assert(MaxHeapSize % max_alignment() == 0, "maximum heap alignment");
	}
	
```


