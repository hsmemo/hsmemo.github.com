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
void G1MarkSweep::mark_sweep_phase2() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この段階では, 全ての live オブジェクトに印(mark)が付いた状態になっている. 
       ここで, 全ての live オブジェクトについて, コンパクション後の新しいアドレスを計算する)
    
      (なお, Perm 領域の処理を最後にする必要がある. 
  
       というのは, 死んだオブジェクトは int 配列で上書きするが, 
       Perm 領域を先に処理してしまうと, 先に klassOop オブジェクトが int 配列で上書きされてしまい, 
       後で Perm 以外の領域でそのクラスのインスタンスを見つけた場合に, 
       そのインスタンスの大きさが分からない, ということになりかねないため)
  
      (なお, phase2, phase3, 及び phase4 で同じ順番でヒープを辿る必要は無いのだが, 
       ValidateMarkSweep ではそれを想定している.
       詳しくは phase4 の下に書いてあるコメント参照.)
      ---------------------------------------- -}

	  // Now all live objects are marked, compute the new object addresses.
	
	  // It is imperative that we traverse perm_gen LAST. If dead space is
	  // allowed a range of dead object may get overwritten by a dead int
	  // array. If perm_gen is not traversed last a klassOop may get
	  // overwritten. This is fine since it is dead, but if the class has dead
	  // instances we have to skip them, and in order to find their size we
	  // need the klassOop!
	  //
	  // It is not required that we traverse spaces in the same order in
	  // phase2, phase3 and phase4, but the ValidateMarkSweep live oops
	  // tracking expects us to do so. See comment under phase4.
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  Generation* pg = g1h->perm_gen();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("2 compute new addresses");
	  TraceTime tm("phase 2", PrintGC && Verbose, true, gclog_or_tty);
	  GenMarkSweep::trace("2");
	
  {- -------------------------------------------
  (1) G1PrepareCompactClosure を引数として
      G1CollectedHeap::heap_region_iterate() を呼び出し, 
      HeapRegion 内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
  
      (なお, コンパクション先の HeapRegion としては, 
       まずは一番先頭の HeapRegion を指定している (これは FindFirstRegionClosure で取得する).
       ただし, 一番先頭の HeapRegion が Humongous でしかも生きていれば, 
       その HeapRegion の next_compaction_space が指す HeapRegion を使用する.)
      ---------------------------------------- -}

	  FindFirstRegionClosure cl;
	  g1h->heap_region_iterate(&cl);
	  HeapRegion *r = cl.result();
	  CompactibleSpace* sp = r;
	  if (r->isHumongous() && oop(r->bottom())->is_gc_marked()) {
	    sp = r->next_compaction_space();
	  }
	
	  G1PrepareCompactClosure blk(sp);
	  g1h->heap_region_iterate(&blk);

  {- -------------------------------------------
  (1) G1PrepareCompactClosure::update_sets() を呼んで, 
      このスレッドの thread local な HumongousRegionSet の結果を
      本体である MasterHumongousRegionSet に反映させる.
  
      (より正確には, blk 変数に束縛されている G1PrepareCompactClosure オブジェクト内の 
       HumongousRegionSet (_humongous_proxy_set フィールド) を, 
       G1CollectedHeap::_humongous_set に格納されている MasterHumongousRegionSet に反映させる)
      ---------------------------------------- -}

	  blk.update_sets();
	
  {- -------------------------------------------
  (1) Generation::prepare_for_compaction() を呼んで, 
      Perm 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
      ---------------------------------------- -}

	  CompactPoint perm_cp(pg, NULL, NULL);
	  pg->prepare_for_compaction(&perm_cp);
	}
	
```


