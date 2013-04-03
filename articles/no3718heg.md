---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// Used by ParallelScavenge
```

### 名前(function name)
```
void Threads::create_thread_roots_tasks(GCTaskQueue* q) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 各 JavaThread に対応する ThreadRootsTask をキューに追加
      (JavaThread の個数分だけの ThreadRootsTask が登録される)
      ---------------------------------------- -}

	  ALL_JAVA_THREADS(p) {
	    q->enqueue(new ThreadRootsTask(p));
	  }

  {- -------------------------------------------
  (1) 最後に VMThread に対応する ThreadRootsTask をキューに追加.
      ---------------------------------------- -}

	  q->enqueue(new ThreadRootsTask(VMThread::vm_thread()));
	}
	
```


