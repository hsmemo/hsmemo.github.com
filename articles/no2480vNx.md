---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp
### 説明(description)

```
// Failed allocation policy. Must be called from the VM thread, and
// only at a safepoint! Note that this method has policy for allocation
// flow, and NOT collection policy. So we do not check for gc collection
// time over limit here, that is the responsibility of the heap specific
// collection methods. This method decides where to attempt allocations,
// and when to attempt collections, but no collection specific policy.
```

### 名前(function name)
```
HeapWord* ParallelScavengeHeap::failed_mem_allocate(size_t size, bool is_tlab) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "should be at safepoint");
	  assert(Thread::current() == (Thread*)VMThread::vm_thread(), "should be in vm thread");
	  assert(!Universe::heap()->is_gc_active(), "not reentrant");
	  assert(!Heap_lock->owned_by_self(), "this thread should not own the Heap_lock");
	
  {- -------------------------------------------
  (1) (PSScavenge 実行前の mark sweep 実行回数を記録しておく)
      ---------------------------------------- -}

	  size_t mark_sweep_invocation_count = total_invocations();
	
  {- -------------------------------------------
  (1) まず PSScavenge::invoke() で PSScavenge による GC を行う.
      ---------------------------------------- -}

	  // We assume (and assert!) that an allocation at this point will fail
	  // unless we collect.
	
	  // First level allocation failure, scavenge and allocate in young gen.
	  GCCauseSetter gccs(this, GCCause::_allocation_failure);
	  PSScavenge::invoke();

  {- -------------------------------------------
  (1) GC が終わったら, PSYoungGen::allocate() での確保を試みる.
      ---------------------------------------- -}

	  HeapWord* result = young_gen()->allocate(size, is_tlab);
	
  {- -------------------------------------------
  (1) 確保が成功しなければ, invoke_full_gc() で Full GC を行った後, もう一度 PSYoungGen::allocate() での確保を試みる.
      (なお, PSScavenge 内で mark sweep が実行されることがあるため, total_invocations() の値を確認している.
       PSScavenge 実行前と値が違えば, 既に mark sweep が実行されているので (= もう一度 mark sweep しても結果は変わらないので) 
       ここでは何もしない.)
      ---------------------------------------- -}

	  // Second level allocation failure.
	  //   Mark sweep and allocate in young generation.
	  if (result == NULL) {
	    // There is some chance the scavenge method decided to invoke mark_sweep.
	    // Don't mark sweep twice if so.
	    if (mark_sweep_invocation_count == total_invocations()) {
	      invoke_full_gc(false);
	      result = young_gen()->allocate(size, is_tlab);
	    }
	  }
	
  {- -------------------------------------------
  (1) まだ確保が成功していなければ, PSOldGen::allocate() で Old Generation から確保を試みる.
      ---------------------------------------- -}

	  // Third level allocation failure.
	  //   After mark sweep and young generation allocation failure,
	  //   allocate in old generation.
	  if (result == NULL && !is_tlab) {
	    result = old_gen()->allocate(size, is_tlab);
	  }
	
  {- -------------------------------------------
  (1) まだ確保が成功していなければ, invoke_full_gc() で(今度は完璧に) Full GC を行い, 
      その後 PSYoungGen::allocate() での確保を試みる.
      ---------------------------------------- -}

	  // Fourth level allocation failure. We're running out of memory.
	  //   More complete mark sweep and allocate in young generation.
	  if (result == NULL) {
	    invoke_full_gc(true);
	    result = young_gen()->allocate(size, is_tlab);
	  }
	
  {- -------------------------------------------
  (1) それでもまだ確保が成功していなければ, PSOldGen::allocate() でもう一度 Old Generation から確保を試みる.
       (今度は完璧に Full GC した後なので, さっきとは結果が変わる可能性がある)
      ---------------------------------------- -}

	  // Fifth level allocation failure.
	  //   After more complete mark sweep, allocate in old generation.
	  if (result == NULL && !is_tlab) {
	    result = old_gen()->allocate(size, is_tlab);
	  }
	
  {- -------------------------------------------
  (1) 結果をリターンする.
      ---------------------------------------- -}

	  return result;
	}
	
```


