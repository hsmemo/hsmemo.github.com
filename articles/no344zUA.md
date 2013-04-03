---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
### 説明(description)

```
// This method should contain all heap-specific policy for invoking a full
// collection.  invoke_no_policy() will only attempt to compact the heap; it
// will do nothing further.  If we need to bail out for policy reasons, scavenge
// before full gc, or any other specialized behavior, it needs to be added here.
//
// Note that this method should only be called from the vm_thread while at a
// safepoint.
//
// Note that the all_soft_refs_clear flag in the collector policy
// may be true because this method can be called without intervening
// activity.  For example when the heap space is tight and full measure
// are being taken to free space.
```

### 名前(function name)
```
void PSParallelCompact::invoke(bool maximum_heap_compaction) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "should be at safepoint");
	  assert(Thread::current() == (Thread*)VMThread::vm_thread(),
	         "should be in vm thread");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = gc_heap();
	  GCCause::Cause gc_cause = heap->gc_cause();
	  assert(!heap->is_gc_active(), "not reentrant");
	
	  PSAdaptiveSizePolicy* policy = heap->size_policy();
	  IsGCActiveMark mark;
	
  {- -------------------------------------------
  (1) ScavengeBeforeFullGC オプションが指定されていれば, Major GC を実行する前に
      PSScavenge::invoke_no_policy() を呼んで Minor GC を実行しておく.
      ---------------------------------------- -}

	  if (ScavengeBeforeFullGC) {
	    PSScavenge::invoke_no_policy();
	  }
	
  {- -------------------------------------------
  (1) PSParallelCompact::invoke_no_policy() を呼び出す
      ---------------------------------------- -}

	  const bool clear_all_soft_refs =
	    heap->collector_policy()->should_clear_all_soft_refs();
	
	  PSParallelCompact::invoke_no_policy(clear_all_soft_refs ||
	                                      maximum_heap_compaction);
	}
	
```


