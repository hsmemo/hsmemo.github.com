---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp
### 説明(description)

```
// Before delegating the resize to the young generation,
// the reserved space for the young and old generations
// may be changed to accomodate the desired resize.
```

### 名前(function name)
```
void ParallelScavengeHeap::resize_young_gen(size_t eden_size,
    size_t survivor_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) UseAdaptiveGCBoundary オプションが指定されている場合は, 
      最初に「既に領域長が調整済みかどうか」を確認しておく
      (PSAdaptiveSizePolicy::bytes_absorbed_from_eden() の値が非ゼロであれば調整済み.
       これは GC 後にも Eden に live object が残っていたため調整処理が走ったケース.
       (See: PSMarkSweep::absorb_live_data_from_eden(), PSParallelCompact::absorb_live_data_from_eden()))
    
      調整済みであれば, (これ以上の調整は不要なので) ここでリターン.
      (なお, リターンする前に PSAdaptiveSizePolicy::reset_bytes_absorbed_from_eden() を呼んで
       bytes_absorbed_from_eden の値を 0 にリセットしている)
      ---------------------------------------- -}

	  if (UseAdaptiveGCBoundary) {
	    if (size_policy()->bytes_absorbed_from_eden() != 0) {
	      size_policy()->reset_bytes_absorbed_from_eden();
	      return;  // The generation changed size already.
	    }

  {- -------------------------------------------
  (1) UseAdaptiveGCBoundary オプションが指定されているが
      まだ領域長が調整済みでない場合には, 
      AdjoiningGenerations::adjust_boundary_for_young_gen_needs() を呼んで
      必要に応じて young-old 境界を動かしておく.
      ---------------------------------------- -}

	    gens()->adjust_boundary_for_young_gen_needs(eden_size, survivor_size);
	  }
	
  {- -------------------------------------------
  (1) PSYoungGen::resize() を呼び出して, PSYoungGen の領域長を変更する.
      ---------------------------------------- -}

	  // Delegate the resize to the generation.
	  _young_gen->resize(eden_size, survivor_size);
	}
	
```


