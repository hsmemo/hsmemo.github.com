---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp

### 名前(function name)
```
void PSMarkSweep::mark_sweep_phase2() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("2 compute new addresses");
	  TraceTime tm("phase 2", PrintGCDetails && Verbose, true, gclog_or_tty);
	  trace("2");
	
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
       詳しくは phase4 の下に書いてあるコメント参照.
       <= と書いてあるんだが, ここのコメントは GenMarkSweep::mark_sweep_phase2() からコピペされてきたもので, 正しくない.
          少なくとも PSMarkSweep 関係の phase4 処理中にはそういうコメントはない.
          GenMarkSweep::mark_sweep_phase4() 内には ValidateMarkSweep に関するコメントが書かれている.)
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

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	
	  PSOldGen* old_gen = heap->old_gen();
	  PSPermGen* perm_gen = heap->perm_gen();
	
  {- -------------------------------------------
  (1) PSOldGen::precompact() を呼んで, 
      Old 領域および New 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
    
      (なお, PSMarkSweepDecorator::set_destination_decorator_tenured() を呼んだ後で
       PSOldGen::precompact() が呼び出されているため, 
       新しいアドレスは Old 領域をコンパクション先として計算される)
      ---------------------------------------- -}

	  // Begin compacting into the old gen
	  PSMarkSweepDecorator::set_destination_decorator_tenured();
	
	  // This will also compact the young gen spaces.
	  old_gen->precompact();
	
  {- -------------------------------------------
  (1) PSPermGen::precompact() を呼んで, 
      Perm 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
  
      (なお, PSMarkSweepDecorator::set_destination_decorator_perm_gen() を呼んだ後で
       PSPermGen::precompact() が呼び出されているため, 
       新しいアドレスは Perm 領域をコンパクション先として計算される)
      ---------------------------------------- -}

	  // Compact the perm gen into the perm gen
	  PSMarkSweepDecorator::set_destination_decorator_perm_gen();
	
	  perm_gen->precompact();
	}
	
```


