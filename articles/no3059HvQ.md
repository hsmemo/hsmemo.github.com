---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp

### 名前(function name)
```
void ThreadLocalAllocBuffer::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::initialize(HeapWord* start, HeapWord* top, HeapWord* end) を呼んで, 
      フィールドの初期化を行う.
      ---------------------------------------- -}

	  initialize(NULL,                    // start
	             NULL,                    // top
	             NULL);                   // end
	
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::set_desired_size() を呼んで, 
      ThreadLocalAllocBuffer::desired_size() の値を初期化しておく.
      ---------------------------------------- -}

	  set_desired_size(initial_desired_size());
	
  {- -------------------------------------------
  (1) _allocation_fraction に入っている統計情報を初期化する.
    
      (これは TLAB の動的サイズ調整時に参照される情報 (See: ThreadLocalAllocBuffer::resize()))
  
      (ただし, メインスレッド(primordial thread) の場合は, Universe の初期化が終わる前にここに到達する.
       その場合だけは, 初期化に必要な情報が得られないため, ここでは初期化しない.
       メインスレッドについては, 代わりに ThreadLocalAllocBuffer::startup_initialization() で初期化する.)
      ---------------------------------------- -}

	  // Following check is needed because at startup the main (primordial)
	  // thread is initialized before the heap is.  The initialization for
	  // this thread is redone in startup_initialization below.
	  if (Universe::heap() != NULL) {
	    size_t capacity   = Universe::heap()->tlab_capacity(myThread()) / HeapWordSize;
	    double alloc_frac = desired_size() * target_refills() / (double) capacity;
	    _allocation_fraction.sample(alloc_frac);
	  }
	
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::refill_waste_limit() の値を初期値に戻しておく.
      (See: CollectedHeap::allocate_from_tlab_slow())
      ---------------------------------------- -}

	  set_refill_waste_limit(initial_refill_waste_limit());
	
  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::initialize_statistics() を呼んで, 
      統計情報を全て 0 にリセットしておく
      ---------------------------------------- -}

	  initialize_statistics();
	}
	
```


