---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/satbQueue.cpp

### 名前(function name)
```
void SATBMarkQueueSet::handle_zero_index_for_thread(JavaThread* t) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  DEBUG_ONLY(t->satb_mark_queue().verify_oops_in_buffer();)

  {- -------------------------------------------
  (1) PtrQueue::handle_zero_index() を呼び出すだけ.
      ---------------------------------------- -}

	  t->satb_mark_queue().handle_zero_index();
	}
	
```


