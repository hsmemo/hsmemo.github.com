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
void GCTaskManager::print_task_time_stamps() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 全ての GCTaskThread に対して, GCTaskThread::print_task_time_stamps() を呼び出すだけ.
      ---------------------------------------- -}

	  for(uint i=0; i<ParallelGCThreads; i++) {
	    GCTaskThread* t = thread(i);
	    t->print_task_time_stamps();
	  }
	}
	
```


