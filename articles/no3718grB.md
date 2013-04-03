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
void GCTaskManager::add_list(GCTaskQueue* list) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(list != NULL, "shouldn't have null task");

  {- -------------------------------------------
  (1) (以下の処理は "GCTaskManager monitor" のロックを取った状態で行う)
      ---------------------------------------- -}

	  MutexLockerEx ml(monitor(), Mutex::_no_safepoint_check_flag);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceGCTaskManager) {
	    tty->print_cr("GCTaskManager::add_list(%u)", list->length());
	  }

  {- -------------------------------------------
  (1) キュー(GCTaskManager::_queue)に GCTask を詰める.
      ---------------------------------------- -}

	  queue()->enqueue(list);

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // Notify with the lock held to avoid missed notifies.
	  if (TraceGCTaskManager) {
	    tty->print_cr("    GCTaskManager::add_list (%s)->notify_all",
	                  monitor()->name());
	  }

  {- -------------------------------------------
  (1) キューに GCTask を追加したので, GCTaskManager::get_task() 等でブロックしているスレッドを起こすために
      "GCTaskManager monitor" に対して notify_all() を呼んでおく.
      ---------------------------------------- -}

	  (void) monitor()->notify_all();

  {- -------------------------------------------
  (1) (MutexLockerEx で取得した "GCTaskManager monitor" のロックは, このブロックを抜けるときにアンロックされる)
      ---------------------------------------- -}

	  // Release monitor().
	}
	
```


