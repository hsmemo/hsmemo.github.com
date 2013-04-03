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
void ThreadRootsTask::do_it(GCTaskManager* manager, uint which) {
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

	  PSPromotionManager* pm = PSPromotionManager::gc_thread_promotion_manager(which);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  PSScavengeRootsClosure roots_closure(pm);
	  CodeBlobToOopClosure roots_in_blobs(&roots_closure, /*do_marking=*/ true);
	
  {- -------------------------------------------
  (1) 処理対象のスレッドが JavaThread であれば JavaThread::oops_do() を呼び出す.
      処理対象のスレッドが VMThread であれば VMThread::oops_do() を呼び出す.
      ---------------------------------------- -}

	  if (_java_thread != NULL)
	    _java_thread->oops_do(&roots_closure, &roots_in_blobs);
	
	  if (_vm_thread != NULL)
	    _vm_thread->oops_do(&roots_closure, &roots_in_blobs);
	
  {- -------------------------------------------
  (1) PSPromotionManager::drain_stacks() で, 
      タスクキューに追加したポインタに対して, 再帰的な Scavenge 処理を行う.
      ---------------------------------------- -}

	  // Do the real work
	  pm->drain_stacks(false);
	}
	
```


