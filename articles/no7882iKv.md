---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/heapInspection.cpp

### 名前(function name)
```
void HeapInspection::heap_inspection(outputStream* st, bool need_prologue) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;
	  HeapWord* ref;
	
	  CollectedHeap* heap = Universe::heap();
	  bool is_shared_heap = false;

  {- -------------------------------------------
  (1) ヒープを管理するクラスに応じて, 以下の局所変数の値を調整しておく.
      * is_shared_heap : 
        SharedHeap のサブクラスであれば true (より具体的には G1CollectedHeap や GenCollectedHeap なら true)
      * ref
        Perm 領域の使用箇所の先頭アドレスに設定.
  
      また, G1CollectedHeap や GenCollectedHeap の場合は, 
      need_prologue 引数が true であれば SharedHeap::gc_prologue() の呼び出しも行っておく.
      ---------------------------------------- -}

	  switch (heap->kind()) {
	    case CollectedHeap::G1CollectedHeap:
	    case CollectedHeap::GenCollectedHeap: {
	      is_shared_heap = true;
	      SharedHeap* sh = (SharedHeap*)heap;
	      if (need_prologue) {
	        sh->gc_prologue(false /* !full */); // get any necessary locks, etc.
	      }
	      ref = sh->perm_gen()->used_region().start();
	      break;
	    }
	#ifndef SERIALGC
	    case CollectedHeap::ParallelScavengeHeap: {
	      ParallelScavengeHeap* psh = (ParallelScavengeHeap*)heap;
	      ref = psh->perm_gen()->object_space()->used_region().start();
	      break;
	    }
	#endif // SERIALGC
	    default:
	      ShouldNotReachHere(); // Unexpected heap kind for this op
	  }

  {- -------------------------------------------
  (1) (以下の処理で, ヒープ内のオブジェクトの情報を集める)
      ---------------------------------------- -}

	  // Collect klass instance info
	  KlassInfoTable cit(KlassInfoTable::cit_size, ref);

  {- -------------------------------------------
  (1) (ただし, KlassInfoTable オブジェクトの確保に失敗した場合は情報収集しない)
      ---------------------------------------- -}

	  if (!cit.allocation_failed()) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    // Iterate over objects in the heap
	    RecordInstanceClosure ric(&cit);

    {- -------------------------------------------
  (1.1) CollectedHeap::object_iterate() (をサブクラスがオーバーライドしたもの) を呼び出し, 
        ヒープ中の全てのオブジェクトに対して RecordInstanceClosure を適用する.
        (これにより, KlassInfoTable オブジェクト(cit)中に情報が収集される)
        ---------------------------------------- -}

	    // If this operation encounters a bad object when using CMS,
	    // consider using safe_object_iterate() which avoids perm gen
	    // objects that may contain bad references.
	    Universe::heap()->object_iterate(&ric);
	
    {- -------------------------------------------
  (1.1) RecordInstanceClosure::missed_count() を呼んで, 
        途中で C ヒープ不足にならなかったかどうかを確認しておく.
        C ヒープ不足だった場合は, そのことを出力内容に追加.
        ---------------------------------------- -}

	    // Report if certain classes are not counted because of
	    // running out of C-heap for the histogram.
	    size_t missed_count = ric.missed_count();
	    if (missed_count != 0) {
	      st->print_cr("WARNING: Ran out of C-heap; undercounted " SIZE_FORMAT
	                   " total instances in data below",
	                   missed_count);
	    }

    {- -------------------------------------------
  (1.1) (変数宣言など)  
        ---------------------------------------- -}

	    // Sort and print klass instance info
	    KlassInfoHisto histo("\n"
	                     " num     #instances         #bytes  class name\n"
	                     "----------------------------------------------",
	                     KlassInfoHisto::histo_initial_size);
	    HistoClosure hc(&histo);

    {- -------------------------------------------
  (1.1) HistoClosure オブジェクト(hc) によって, 
        収集した内容を KlassInfoHisto オブジェクト(histo)内に移し替える.
        ---------------------------------------- -}

	    cit.iterate(&hc);

    {- -------------------------------------------
  (1.1) ヒストグラムをソートしておく.
        ---------------------------------------- -}

	    histo.sort();

    {- -------------------------------------------
  (1.1) 結果を出力内容に追加
        ---------------------------------------- -}

	    histo.print_on(st);
	  } else {

  {- -------------------------------------------
  (1) (なお, KlassInfoTable の確保に失敗した場合は「C ヒープが不足している」というメッセージだけを出力に追加)
      ---------------------------------------- -}

	    st->print_cr("WARNING: Ran out of C-heap; histogram not generated");
	  }

  {- -------------------------------------------
  (1) 以上の内容を出力する
      ---------------------------------------- -}

	  st->flush();
	
  {- -------------------------------------------
  (1) SharedHeap::gc_prologue() の呼び出しを行った場合は
      (= G1CollectedHeap や GenCollectedHeap で, かつ need_prologue 引数が true の場合は), 
      SharedHeap::gc_epilogue() を呼び出しておく.
      ---------------------------------------- -}

	  if (need_prologue && is_shared_heap) {
	    SharedHeap* sh = (SharedHeap*)heap;
	    sh->gc_epilogue(false /* !full */); // release all acquired locks, etc.
	  }
	}
	
```


