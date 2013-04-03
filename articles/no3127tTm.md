---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.cpp

### 名前(function name)
```
HeapWord*
HeapRegion::
oops_on_card_seq_iterate_careful(MemRegion mr,
                                 FilterOutOfRegionClosure* cl,
                                 bool filter_young,
                                 jbyte* card_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Currently, we should only have to clean the card if filter_young
	  // is true and vice versa.
	  if (filter_young) {
	    assert(card_ptr != NULL, "pre-condition");
	  } else {
	    assert(card_ptr == NULL, "pre-condition");
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	
  {- -------------------------------------------
  (1) mr 引数の値を少し調整(intersection() を呼んで, 実際に使用中の領域だけに限定する)
      結果として処理する領域がなくなってしまったら, ここでリターン.
      ---------------------------------------- -}

	  // If we're within a stop-world GC, then we might look at a card in a
	  // GC alloc region that extends onto a GC LAB, which may not be
	  // parseable.  Stop such at the "saved_mark" of the region.
	  if (G1CollectedHeap::heap()->is_gc_active()) {
	    mr = mr.intersection(used_region_at_save_marks());
	  } else {
	    mr = mr.intersection(used_region());
	  }
	  if (mr.is_empty()) return NULL;
	  // Otherwise, find the obj that extends onto mr.start().
	
  {- -------------------------------------------
  (1) もしこの HeapRegion が Young の場合,
      filter_young 引数が true なら, することはない.
      ここでリターン.
      ---------------------------------------- -}

	  // The intersection of the incoming mr (for the card) and the
	  // allocated part of the region is non-empty. This implies that
	  // we have actually allocated into this region. The code in
	  // G1CollectedHeap.cpp that allocates a new region sets the
	  // is_young tag on the region before allocating. Thus we
	  // safely know if this region is young.
	  if (is_young() && filter_young) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!is_young(), "check value of filter_young");
	
  {- -------------------------------------------
  (1) card_ptr 引数が示す場所に CardTableModRefBS::clean_card_val() を書き込んでおく.
      メモリバリアもはっておく.
      (NULL なら何もしないが...)
  
      (これ以降に同じ card 内の領域が変更されたときにちゃんと検出できるように
      card table 中の対応する値を clean に戻す処理だと思われる)
      ---------------------------------------- -}

	  // We can only clean the card here, after we make the decision that
	  // the card is not young. And we only clean the card if we have been
	  // asked to (i.e., card_ptr != NULL).
	  if (card_ptr != NULL) {
	    *card_ptr = CardTableModRefBS::clean_card_val();
	    // We must complete this write before we do any of the reads below.
	    OrderAccess::storeload();
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)  &(assert)
      ---------------------------------------- -}

	  // We used to use "block_start_careful" here.  But we're actually happy
	  // to update the BOT while we do this...
	  HeapWord* cur = block_start(mr.start());
	  assert(cur <= mr.start(), "Postcondition");
	
  {- -------------------------------------------
  (1) start 地点をまたがる(or start 地点からちょうど始まる)オブジェクトを探す.
      (block_start() 地点(= cur の初期値)から開始し,
       終端が start 地点を越える最初のオブジェクトが見つかるまでポインタを進める.)
      ---------------------------------------- -}

	  while (cur <= mr.start()) {
	    if (oop(cur)->klass_or_null() == NULL) {
	      // Ran into an unparseable point.
	      return cur;
	    }
	    // Otherwise...
	    int sz = oop(cur)->size();
	    if (cur + sz > mr.start()) break;
	    // Otherwise, go on.
	    cur = cur + sz;
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  oop obj;
	  obj = oop(cur);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // If we finish this loop...
	  assert(cur <= mr.start()
	         && obj->klass_or_null() != NULL
	         && cur + obj->size() > mr.start(),
	         "Loop postcondition");

  {- -------------------------------------------
  (1) 見つけたオブジェクトが生きていれば
      oop_iterate() で cl 引数の FilterOutOfRegionClosure を適用.
      (なお mr を指定して, end 以降まで伸びている場合でも end で打ちきらせる)
      ---------------------------------------- -}

	  if (!g1h->is_obj_dead(obj)) {
	    obj->oop_iterate(cl, mr);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HeapWord* next;

  {- -------------------------------------------
  (1) 見つけたオブジェクト以降の生きているオブジェクトに付いても,
      oop_iterate() を適用していき,
      終わったらリターン.
  
      (なお, end 移行まではみ出ている配列オブジェクトに付いては
       mr を指定して, end で打ちきらせている)
  
      以下のどちらかが成り立てば処理は終了.
      * end までの範囲をすべて処理し終わる
        この場合は, NULL をリターン
      * 途中で klass_or_null() が NULL のオブジェクト(unparseable な地点)が見つかる.
        この場合は, その地点のアドレスをリターン.
      ---------------------------------------- -}

	  while (cur < mr.end()) {
	    obj = oop(cur);
	    if (obj->klass_or_null() == NULL) {
	      // Ran into an unparseable point.
	      return cur;
	    };
	    // Otherwise:
	    next = (cur + obj->size());
	    if (!g1h->is_obj_dead(obj)) {
	      if (next < mr.end()) {
	        obj->oop_iterate(cl);
	      } else {
	        // this obj spans the boundary.  If it's an array, stop at the
	        // boundary.
	        if (obj->is_objArray()) {
	          obj->oop_iterate(cl, mr);
	        } else {
	          obj->oop_iterate(cl);
	        }
	      }
	    }
	    cur = next;
	  }
	  return NULL;
	}
	
```


