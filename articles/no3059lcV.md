---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vmThread.cpp

### 名前(function name)
```
VMOperationQueue::VMOperationQueue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // The queue is a circular doubled-linked list, which always contains
	  // one element (i.e., one element means empty).
	  for(int i = 0; i < nof_priorities; i++) {
	    _queue_length[i] = 0;
	    _queue_counter = 0;
	    _queue[i] = new VM_Dummy();
	    _queue[i]->set_next(_queue[i]);
	    _queue[i]->set_prev(_queue[i]);
	  }
	  _drain_list = NULL;
	}
	
```


