---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/dirtyCardQueue.cpp

### 名前(function name)
```
bool DirtyCardQueueSet::mut_process_buffer(void** buf) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Used to determine if we had already claimed a par_id
	  // before entering this method.
	  bool already_claimed = false;
	
	  // We grab the current JavaThread.
	  JavaThread* thread = JavaThread::current();
	
	  // We get the the number of any par_id that this thread
	  // might have already claimed.
	  int worker_i = thread->get_claimed_par_id();
	
	  // If worker_i is not -1 then the thread has already claimed
	  // a par_id. We make note of it using the already_claimed value
	  if (worker_i != -1) {
	    already_claimed = true;
	  } else {
	
	    // Otherwise we need to claim a par id
	    worker_i = _free_ids->claim_par_id();
	
	    // And store the par_id value in the thread
	    thread->set_claimed_par_id(worker_i);
	  }
	
	  bool b = false;
	  if (worker_i != -1) {
	    b = DirtyCardQueue::apply_closure_to_buffer(_closure, buf, 0,
	                                                _sz, true, worker_i);
	    if (b) Atomic::inc(&_processed_buffers_mut);
	
	    // If we had not claimed an id before entering the method
	    // then we must release the id.
	    if (!already_claimed) {
	
	      // we release the id
	      _free_ids->release_par_id(worker_i);
	
	      // and set the claimed_id in the thread to -1
	      thread->set_claimed_par_id(-1);
	    }
	  }
	  return b;
	}
	
```


