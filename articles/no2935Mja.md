---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp
### 説明(description)

```
// Checkpoint the roots into this generation from outside
// this generation. [Note this initial checkpoint need only
// be approximate -- we'll do a catch up phase subsequently.]
```

### 名前(function name)
```
void ConcurrentMark::checkpointRootsInitial() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "world should be stopped");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (処理の開始時間を記録しておく)
      ---------------------------------------- -}

	  double start = os::elapsedTime();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  G1CollectorPolicy* g1p = G1CollectedHeap::heap()->g1_policy();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1p->record_concurrent_mark_init_start();
	  checkpointRootsInitialPre();
	
	  // YSR: when concurrent precleaning is in place, we'll
	  // need to clear the cached card table here
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;
	  HandleMark  hm;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1h->ensure_parsability(false);
	  g1h->perm_gen()->save_marks();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  CMMarkRootsClosure notOlder(this, g1h, false);
	  CMMarkRootsClosure older(this, g1h, true);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1h->set_marking_started();
	  g1h->rem_set()->prepare_for_younger_refs_iterate(false);
	
  {- -------------------------------------------
  (1) CMMarkRootsClosure を引数として SharedHeap::process_strong_roots() を呼び出し, 
      strong root から辿り着けるオブジェクト全てに mark を付ける.
      ---------------------------------------- -}

	  g1h->process_strong_roots(true,    // activate StrongRootsScope
	                            false,   // fake perm gen collection
	                            SharedHeap::SO_AllClasses,
	                            &notOlder, // Regular roots
	                            NULL,     // do not visit active blobs
	                            &older    // Perm Gen Roots
	                            );

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  checkpointRootsInitialPost();
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Statistics.
	  double end = os::elapsedTime();
	  _init_times.add((end - start) * 1000.0);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  g1p->record_concurrent_mark_init_end();
	}
	
```


