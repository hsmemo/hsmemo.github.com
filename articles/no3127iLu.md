---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
  bool do_bit(size_t offset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    HeapWord* addr = _nextMarkBitMap->offsetToHeapWord(offset);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(_nextMarkBitMap->isMarked(addr), "invariant");
	    assert( addr < _cm->finger(), "invariant");
	
  {- -------------------------------------------
  (1) finger を更新している?? (#TODO)
      ---------------------------------------- -}

	    if (_scanning_heap_region) {
	      statsOnly( _task->increase_objs_found_on_bitmap() );
	      assert(addr >= _task->finger(), "invariant");
	      // We move that task's local finger along.
	      _task->move_finger_to(addr);
	    } else {
	      // We move the task's region finger along.
	      _task->move_region_finger_to(addr);
	    }
	
  {- -------------------------------------------
  (1) CMTask::scan_object() を呼んで処理を行う.
      ---------------------------------------- -}

	    _task->scan_object(oop(addr));
	    // we only partially drain the local queue and global stack
	    _task->drain_local_queue(true);
	    _task->drain_global_stack(true);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // if the has_aborted flag has been raised, we need to bail out of
	    // the iteration
	    return !_task->has_aborted();
	  }
	
```


