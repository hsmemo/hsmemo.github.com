---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp

### 名前(function name)
```
void G1MarkSweep::mark_sweep_phase3() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  Generation* pg = g1h->perm_gen();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // Adjust the pointers to reflect the new locations
	  EventMark m("3 adjust pointers");
	  TraceTime tm("phase 3", PrintGC && Verbose, true, gclog_or_tty);
	  GenMarkSweep::trace("3");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  SharedHeap* sh = SharedHeap::heap();
	
  {- -------------------------------------------
  (1) まず, strong root 内に格納されているポインタの修正処理を行う.
      ---------------------------------------- -}

	  sh->process_strong_roots(true,  // activate StrongRootsScope
	                           true,  // Collecting permanent generation.
	                           SharedHeap::SO_AllClasses,
	                           &GenMarkSweep::adjust_root_pointer_closure,
	                           NULL,  // do not touch code cache here
	                           &GenMarkSweep::adjust_pointer_closure);
	
	  g1h->ref_processor()->weak_oops_do(&GenMarkSweep::adjust_root_pointer_closure);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Now adjust pointers in remaining weak roots.  (All of which should
	  // have been cleared if they pointed to non-surviving objects.)
	  g1h->g1_process_weak_roots(&GenMarkSweep::adjust_root_pointer_closure,
	                             &GenMarkSweep::adjust_pointer_closure);
	
  {- -------------------------------------------
  (1) MarkSweep::adjust_marks() を呼んで, 
      MarkSweep::preserve_mark() で待避された mark 値内のポインタについても, 値を修正しておく.
      (See: PreservedMark)
      ---------------------------------------- -}

	  GenMarkSweep::adjust_marks();
	
  {- -------------------------------------------
  (1) G1CollectedHeap::heap_region_iterate() を呼んで, HeapRegion 内のポインタを修正する.
      ---------------------------------------- -}

	  G1AdjustPointersClosure blk;
	  g1h->heap_region_iterate(&blk);

  {- -------------------------------------------
  (1) CompactingPermGenGen::adjust_pointers() で, Perm 領域のポインタを修正する.
      ---------------------------------------- -}

	  pg->adjust_pointers();
	}
	
```


