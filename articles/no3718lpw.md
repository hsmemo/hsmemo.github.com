---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.cpp

### 名前(function name)
```
void PSPromotionManager::drain_stacks_depth(bool totally_drain) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  totally_drain = totally_drain || _totally_drain;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	  MutableSpace* to_space = heap->young_gen()->to_space();
	  MutableSpace* old_space = heap->old_gen()->object_space();
	  MutableSpace* perm_space = heap->perm_gen()->object_space();
	#endif /* ASSERT */
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OopStarTaskQueue* const tq = claimed_stack_depth();

  {- -------------------------------------------
  (1) (以下の do...while ブロック内で, 
       PSPromotionManager が使用するタスクキュー(claimed_stack_depth())内のポインタの処理を行う)
  
      (この do...while ループ自体は, 以下の条件が満たされた時に終了する.
       * 引数の totally_drain が true であるか, _totally_drain(See: PSPromotionManager::PSPromotionManager()) が true の場合
         タスクキューが完全に空になれば終了.
       * そうではない場合
         タスクキューの overflow stack(See: OverflowTaskQueue) が空になっていれば終了.)
      ---------------------------------------- -}

	  do {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    StarTask p;
	
    {- -------------------------------------------
  (1.1) まず, タスクキューの overflow stack 内の全ポインタを process_popped_location_depth() で処理する.
        ---------------------------------------- -}

	    // Drain overflow stack first, so other threads can steal from
	    // claimed stack while we work.
	    while (tq->pop_overflow(p)) {
	      process_popped_location_depth(p);
	    }
	
    {- -------------------------------------------
  (1.1) 次に, タスクキュー内のポインタを process_popped_location_depth() で処理する.
        * 引数の totally_drain が true であるか, _totally_drain(See: PSPromotionManager::PSPromotionManager()) が true の場合
          タスクキュー内のポインタを全て処理
        * そうではない場合
          タスクキュー内のポインタ数が _target_stack_size(See: PSPromotionManager::PSPromotionManager()) になるまで処理
        ---------------------------------------- -}

	    if (totally_drain) {
	      while (tq->pop_local(p)) {
	        process_popped_location_depth(p);
	      }
	    } else {
	      while (tq->size() > _target_stack_size && tq->pop_local(p)) {
	        process_popped_location_depth(p);
	      }
	    }
	  } while (totally_drain && !tq->taskqueue_empty() || !tq->overflow_empty());
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!totally_drain || tq->taskqueue_empty(), "Sanity");
	  assert(totally_drain || tq->size() <= _target_stack_size, "Sanity");
	  assert(tq->overflow_empty(), "Sanity");
	}
	
```


