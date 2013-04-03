---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.cpp


### 本体部(body)
```
	void VM_ParallelGCFailedAllocation::doit() {

  {- -------------------------------------------
  (1) (DTrace のフック点) (JVMTI のフック点)
      (See: SvcGCMarker)
      ---------------------------------------- -}

	  SvcGCMarker sgcm(SvcGCMarker::MINOR);
	
  {- -------------------------------------------
  (1) ParallelScavengeHeap::failed_mem_allocate() を呼び出す.
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "must be a ParallelScavengeHeap");
	
	  GCCauseSetter gccs(heap, _gc_cause);
	  _result = heap->failed_mem_allocate(_size, _is_tlab);
	
  {- -------------------------------------------
  (1) もし GC_locker のせいで成功しなかった場合は, _gc_locked フィールドを true にしておく.
      (See: _gc_locked)
      ---------------------------------------- -}

	  if (_result == NULL && GC_locker::is_active_and_needs_gc()) {
	    set_gc_locked();
	  }
	}
	
```


