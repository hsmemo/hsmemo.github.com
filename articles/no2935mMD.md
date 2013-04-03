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
void G1MarkSweep::mark_sweep_phase4() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この段階では, 全てのポインタはコンパクション後の新しいアドレスを指しているはず)
       
      (この phase4 で実際にオブジェクトを移動させる際には, まず Perm 領域から処理を行う.
       これは, instance の処理の際に class オブジェクト(klassOop)が正しい位置にあることを保証したいため)
  
      (なお, ValidateMarkSweep によるチェックでは
       phase2, phase3, 及び phase4 で同じ順番でヒープを辿ると想定しているが, 
       現在の処理ではそうなっていない (phase4 では Perm を先に処理する).
       このため, Perm の verify 処理時には phase2 で記録した higher index を使用するように伝えている.)
      ---------------------------------------- -}

	  // All pointers are now adjusted, move objects accordingly
	
	  // It is imperative that we traverse perm_gen first in phase4. All
	  // classes must be allocated earlier than their instances, and traversing
	  // perm_gen first makes sure that all klassOops have moved to their new
	  // location before any instance does a dispatch through it's klass!
	
	  // The ValidateMarkSweep live oops tracking expects us to traverse spaces
	  // in the same order in phase2, phase3 and phase4. We don't quite do that
	  // here (perm_gen first rather than last), so we tell the validate code
	  // to use a higher index (saved from phase2) when verifying perm_gen.

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  Generation* pg = g1h->perm_gen();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("4 compact heap");
	  TraceTime tm("phase 4", PrintGC && Verbose, true, gclog_or_tty);
	  GenMarkSweep::trace("4");
	
  {- -------------------------------------------
  (1) Generation::compact() で, Perm 領域内のオブジェクトを移動させる.
      ---------------------------------------- -}

	  pg->compact();
	
  {- -------------------------------------------
  (1) G1SpaceCompactClosure を引数として G1CollectedHeap::heap_region_iterate() を呼び出し, 
      各 HeapRegion 内のオブジェクトを移動させる.
      ---------------------------------------- -}

	  G1SpaceCompactClosure blk;
	  g1h->heap_region_iterate(&blk);
	
	}
	
```


