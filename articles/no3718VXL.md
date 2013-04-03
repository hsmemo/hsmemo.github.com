---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psTasks.cpp

### 名前(function name)
```
void StealTask::do_it(GCTaskManager* manager, uint which) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Universe::heap()->is_gc_active(), "called outside gc");
	
  {- -------------------------------------------
  (1) PSPromotionManager::gc_thread_promotion_manager() で, 
      これを実行している GCTaskThread 用の PSPromotionManager オブジェクトを取得する.
      ---------------------------------------- -}

	  PSPromotionManager* pm =
	    PSPromotionManager::gc_thread_promotion_manager(which);

  {- -------------------------------------------
  (1) PSPromotionManager::drain_stacks() で, 
      自分のタスクキューが空になるまで処理を行う.
      ---------------------------------------- -}

	  pm->drain_stacks(true);
	  guarantee(pm->stacks_empty(),
	            "stacks should be empty at this point");
	
  {- -------------------------------------------
  (1) (自分の担当分は終わったので, 以下の while ループで, 
       未処理の仕事が存在しなくなるまで 他の GCTaskThread から仕事を奪って実行する.)
  
      PSPromotionManager::steal_depth() による work stealing を行い, 成功すれば
      奪ってきたポインタに対して PSPromotionManager::process_popped_location_depth() でコピー処理を行い, 
      さらにそこからの参照先に対しても PSPromotionManager::drain_stacks_depth() で再帰的に処理を行う.
  
      (ループが終了する条件は, 
       PSPromotionManager::steal_depth() による work stealing が失敗し, 
       さらに ParallelTaskTerminator::offer_termination() が true を返した場合.)
      ---------------------------------------- -}

	  int random_seed = 17;
	  while(true) {
	    StarTask p;
	    if (PSPromotionManager::steal_depth(which, &random_seed, p)) {
	      TASKQUEUE_STATS_ONLY(pm->record_steal(p));
	      pm->process_popped_location_depth(p);
	      pm->drain_stacks_depth(true);
	    } else {
	      if (terminator()->offer_termination()) {
	        break;
	      }
	    }
	  }
	  guarantee(pm->stacks_empty(), "stacks should be empty at this point");
	}
	
```


