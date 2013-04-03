---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.cpp

### 名前(function name)
```
HeapWord* CollectedHeap::allocate_from_tlab_slow(Thread* thread, size_t size) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) もし現在の TLAB 内に空き領域がかなり残っていれば, それを捨てるのはもったいないので
      今回は TLAB を使わずに slow-path で確保することにする.
      この場合, ここでリターン.
        (なお, 同じサイズの確保を何度も行うようなプログラムの場合, 閾値が同じだと
         ずっとこれに引っかかって slow-path になり続ける恐れがあるので, 
         ThreadLocalAllocBuffer::record_slow_allocation() で
         ThreadLocalAllocBuffer::refill_waste_limit() の値を増加させている.
         See: ThreadLocalAllocBuffer::record_slow_allocation())
      ---------------------------------------- -}

	  // Retain tlab and allocate object in shared space if
	  // the amount free in the tlab is too large to discard.
	  if (thread->tlab().free() > thread->tlab().refill_waste_limit()) {
	    thread->tlab().record_slow_allocation(size);
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::compute_size() で次に確保する TLAB の大きさを計算する.
      ---------------------------------------- -}

	  // Discard tlab and allocate a new one.
	  // To minimize fragmentation, the last TLAB may be smaller than the rest.
	  size_t new_tlab_size = thread->tlab().compute_size(size);
	
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::clear_before_allocation() で, 
      現在の TLAB の残りのスペースを dummy の配列で埋めておく.
      ---------------------------------------- -}

	  thread->tlab().clear_before_allocation();
	
  {- -------------------------------------------
  (1) もし確保する TLAB の大きさが 0 だったら, ここでリターン.
      ---------------------------------------- -}

	  if (new_tlab_size == 0) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) CollectedHeap::allocate_new_tlab() で新しい TLAB を確保する (以下の obj).
  　    (実際にはこのメソッドはそれぞれのサブクラスでオーバーライドされており, 各 GC アルゴリズムに対応した確保処理が行われる)
     もし確保に失敗したら NULL をリターン.
      ---------------------------------------- -}

	  // Allocate a new TLAB...
	  HeapWord* obj = Universe::heap()->allocate_new_tlab(new_tlab_size);
	  if (obj == NULL) {
	    return NULL;
	  }

  {- -------------------------------------------
  (1) ZeroTLAB オプションが指定されていれば, 確保した TLAB 内全域を 0 クリアする.
      そうでなければ, 今回の確保要求に対して返却するオブジェクト分の領域だけを 0 クリアする.
      ---------------------------------------- -}

	  if (ZeroTLAB) {
	    // ..and clear it.
	    Copy::zero_to_words(obj, new_tlab_size);
	  } else {
	    // ...and clear just the allocated object.
	    Copy::zero_to_words(obj, size);
	  }

  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::fill() を呼んで, 確保した TLAB をカレントスレッドの _tlab にセットする.
      (なお, 新しい TLAB は obj から new_tlab_size 分の領域.
       ただし, obj から obj + size までの領域は, 確保要求のあったオブジェクトを配置するために使うため, 
       この時点で既に残り容量は new_tlab_size - size となっている.)
      ---------------------------------------- -}

	  thread->tlab().fill(obj, obj + size, new_tlab_size);

  {- -------------------------------------------
  (1) 確保したオブジェクトへのポインタをリターンする
      ---------------------------------------- -}

	  return obj;
	}
	
```


