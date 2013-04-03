---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/satbQueue.cpp
### 説明(description)

```
// This method removes entries from an SATB buffer that will not be
// useful to the concurrent marking threads. An entry is removed if it
// satisfies one of the following conditions:
//
// * it points to an object outside the G1 heap (G1's concurrent
//     marking only visits objects inside the G1 heap),
// * it points to an object that has been allocated since marking
//     started (according to SATB those objects do not need to be
//     visited during marking), or
// * it points to an object that has already been marked (no need to
//     process it again).
//
// The rest of the entries will be retained and are compacted towards
// the top of the buffer. If with this filtering we clear a large
// enough chunk of the buffer we can re-use it (instead of enqueueing
// it) and we can just allow the mutator to carry on executing.

```

### 名前(function name)
```
bool ObjPtrQueue::should_enqueue_buffer() {
```

### 本体部(body)
```
	  assert(_lock == NULL || _lock->owned_by_self(),
	         "we should have taken the lock before calling this");
	
	  // A value of 0 means "don't filter SATB buffers".
	  if (G1SATBBufferEnqueueingThresholdPercent == 0) {
	    return true;
	  }
	
	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	
	  // This method should only be called if there is a non-NULL buffer
	  // that is full.
	  assert(_index == 0, "pre-condition");
	  assert(_buf != NULL, "pre-condition");
	
	  void** buf = _buf;
	  size_t sz = _sz;
	
	  // Used for sanity checking at the end of the loop.
	  debug_only(size_t entries = 0; size_t retained = 0;)
	
	  size_t i = sz;
	  size_t new_index = sz;
	
	  // Given that we are expecting _index == 0, we could have changed
	  // the loop condition to (i > 0). But we are using _index for
	  // generality.
	  while (i > _index) {
	    assert(i > 0, "we should have at least one more entry to process");
	    i -= oopSize;
	    debug_only(entries += 1;)
	    oop* p = (oop*) &buf[byte_index_to_index((int) i)];
	    oop obj = *p;
	    // NULL the entry so that unused parts of the buffer contain NULLs
	    // at the end. If we are going to retain it we will copy it to its
	    // final place. If we have retained all entries we have visited so
	    // far, we'll just end up copying it to the same place.
	    *p = NULL;
	
	    bool retain = g1h->is_obj_ill(obj);
	    if (retain) {
	      assert(new_index > 0, "we should not have already filled up the buffer");
	      new_index -= oopSize;
	      assert(new_index >= i,
	             "new_index should never be below i, as we alwaysr compact 'up'");
	      oop* new_p = (oop*) &buf[byte_index_to_index((int) new_index)];
	      assert(new_p >= p, "the destination location should never be below "
	             "the source as we always compact 'up'");
	      assert(*new_p == NULL,
	             "we should have already cleared the destination location");
	      *new_p = obj;
	      debug_only(retained += 1;)
	    }
	  }
	  size_t entries_calc = (sz - _index) / oopSize;
	  assert(entries == entries_calc, "the number of entries we counted "
	         "should match the number of entries we calculated");
	  size_t retained_calc = (sz - new_index) / oopSize;
	  assert(retained == retained_calc, "the number of retained entries we counted "
	         "should match the number of retained entries we calculated");
	  size_t perc = retained_calc * 100 / entries_calc;
	  bool should_enqueue = perc > (size_t) G1SATBBufferEnqueueingThresholdPercent;
	  _index = new_index;
	
	  return should_enqueue;
	}
	
```


