---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
void ConcurrentMark::clearNextBitmap() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  G1CollectorPolicy* g1p = g1h->g1_policy();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Make sure that the concurrent mark thread looks to still be in
	  // the current cycle.
	  guarantee(cmThread()->during_cycle(), "invariant");
	
	  // We are finishing up the current cycle by clearing the next
	  // marking bitmap and getting it ready for the next cycle. During
	  // this time no other cycle can start. So, let's make sure that this
	  // is the case.
	  guarantee(!g1h->mark_in_progress(), "invariant");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // clear the mark bitmap (no grey objects to start with).
	  // We need to do this in chunks and offer to yield in between
	  // each chunk.
	  HeapWord* start  = _nextMarkBitMap->startWord();
	  HeapWord* end    = _nextMarkBitMap->endWord();
	  HeapWord* cur    = start;
	  size_t chunkSize = M;

  {- -------------------------------------------
  (1) (以下の while ブロック内で _nextMarkBitMap のクリア処理を行う)
      ---------------------------------------- -}

	  while (cur < end) {

    {- -------------------------------------------
  (1.1) CMBitMap::clearRange() を呼んで, chunkSize 分だけの領域をクリアする
        (なお, 終端(end)を超えていたら end で切り詰める)
        ---------------------------------------- -}

	    HeapWord* next = cur + chunkSize;
	    if (next > end)
	      next = end;
	    MemRegion mr(cur,next);
	    _nextMarkBitMap->clearRange(mr);

    {- -------------------------------------------
  (1.1) 次の領域へ
        ---------------------------------------- -}

	    cur = next;

    {- -------------------------------------------
  (1.1) ConcurrentMark::do_yield_check() を呼んでおく.
  
        (この中では, 登録している SuspendibleThreadSet に対して 
         SuspendibleThreadSet::suspend_all() が呼ばれていれば, 待機処理を行う.
         この待機は SuspendibleThreadSet::resume_all() が呼ばれると解ける)
        ---------------------------------------- -}

	    do_yield_check();
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    // Repeat the asserts from above. We'll do them as asserts here to
	    // minimize their overhead on the product. However, we'll have
	    // them as guarantees at the beginning / end of the bitmap
	    // clearing to get some checking in the product.
	    assert(cmThread()->during_cycle(), "invariant");
	    assert(!g1h->mark_in_progress(), "invariant");
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Repeat the asserts from above.
	  guarantee(cmThread()->during_cycle(), "invariant");
	  guarantee(!g1h->mark_in_progress(), "invariant");
	}
	
```


