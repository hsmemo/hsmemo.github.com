---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/memprofiler.cpp

### 名前(function name)
```
void MemProfilerTask::task() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Get thread lock to provide mutual exclusion, and so we can iterate safely over the thread list.
	  MutexLocker mu(Threads_lock);
	  MemProfiler::do_trace();
	}
	
```


