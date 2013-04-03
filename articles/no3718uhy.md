---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.cpp

### 名前(function name)
```
void GCTaskManager::execute_and_wait(GCTaskQueue* list) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 実行する GCTask 群に WaitForBarrierGCTask を追加しておく.
      ---------------------------------------- -}

	  WaitForBarrierGCTask* fin = WaitForBarrierGCTask::create();
	  list->enqueue(fin);

  {- -------------------------------------------
  (1) GCTaskManager::add_list() で, GCTask をキューに追加して GCTaskThread 達に実行させる.
      ---------------------------------------- -}

	  add_list(list);

  {- -------------------------------------------
  (1) WaitForBarrierGCTask::wait_for() で GCTask が全て実行されるまで待機する.
      待機が解けたら, WaitForBarrierGCTask オブジェクトを破棄して終了.
      ---------------------------------------- -}

	  fin->wait_for();
	  // We have to release the barrier tasks!
	  WaitForBarrierGCTask::destroy(fin);
	}
	
```


