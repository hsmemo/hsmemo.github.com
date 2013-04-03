---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void JavaThread::initialize_queues() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数では G1GC 用の初期化処理を行っている)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!SafepointSynchronize::is_at_safepoint(),
	         "we should not be at a safepoint");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ObjPtrQueue& satb_queue = satb_mark_queue();
	  SATBMarkQueueSet& satb_queue_set = satb_mark_queue_set();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // The SATB queue should have been constructed with its active
	  // field set to false.
	  assert(!satb_queue.is_active(), "SATB queue should not be active");
	  assert(satb_queue.is_empty(), "SATB queue should be empty");

  {- -------------------------------------------
  (1) もし現在 ConcurrentMarkingThread が作業中であれば, 
      (その補佐用に SATB write barrier が必要なので)
      ObjPtrQueue::set_active() を呼んで
      このスレッド用の ObjPtrQueue でも SATB write barrier を有効にしておく.
      ---------------------------------------- -}

	  // If we are creating the thread during a marking cycle, we should
	  // set the active field of the SATB queue to true.
	  if (satb_queue_set.is_active()) {
	    satb_queue.set_active(true);
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  DirtyCardQueue& dirty_queue = dirty_card_queue();
	  // The dirty card queue should have been constructed with its
	  // active field set to true.
	  assert(dirty_queue.is_active(), "dirty card queue should be active");
	}
	
```


