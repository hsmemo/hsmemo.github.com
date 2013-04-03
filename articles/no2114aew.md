---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweepDecorator.cpp

### 名前(function name)
```
void PSMarkSweepDecorator::advance_destination_decorator() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (コンパクション先として使用する領域を, 次の領域に切り替える.
  
       具体的には, 現在の領域に応じて, 以下のように切り替わっていく.
            Old -> Eden -> From -> To                       
      )
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	
	  assert(_destination_decorator != NULL, "Sanity");
	  guarantee(_destination_decorator != heap->perm_gen()->object_mark_sweep(), "Cannot advance perm gen decorator");
	
	  PSMarkSweepDecorator* first = heap->old_gen()->object_mark_sweep();
	  PSMarkSweepDecorator* second = heap->young_gen()->eden_mark_sweep();
	  PSMarkSweepDecorator* third = heap->young_gen()->from_mark_sweep();
	  PSMarkSweepDecorator* fourth = heap->young_gen()->to_mark_sweep();
	
	  if ( _destination_decorator == first ) {
	    _destination_decorator = second;
	  } else if ( _destination_decorator == second ) {
	    _destination_decorator = third;
	  } else if ( _destination_decorator == third ) {
	    _destination_decorator = fourth;
	  } else {
	    fatal("PSMarkSweep attempting to advance past last compaction area");
	  }
	}
	
```


