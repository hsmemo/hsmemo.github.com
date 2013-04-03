---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
HeapWord*
G1CollectedHeap::mem_allocate(size_t word_size,
                              bool   is_noref,
                              bool   is_tlab,
                              bool*  gc_overhead_limit_was_exceeded) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_heap_not_locked_and_not_at_safepoint();
	  assert(!is_tlab, "mem_allocate() this should not be called directly "
	         "to allocate TLABs");
	
  {- -------------------------------------------
  (1) 以下, メモリ確保が成功するかあるいは成功しないと判断するまで, 以下のループで処理を続ける
      (以下のループを抜けるのは return する時だけ)
      ---------------------------------------- -}

	  // Loop until the allocation is satisified, or unsatisfied after GC.
	  for (int try_count = 1; /* we'll return */; try_count += 1) {
	    unsigned int gc_count_before;
	
    {- -------------------------------------------
  (1.1) まず, G1CollectedHeap::attempt_allocation() または G1CollectedHeap::attempt_allocation_humongous() で確保を試みる.
        (確保要求のサイズがあまりに大きくなければ (= G1CollectedHeap::isHumongous() が false), G1CollectedHeap::attempt_allocation() を使う.
        大きければ G1CollectedHeap::attempt_allocation_humongous() を使う.)
        
        確保が成功すれば, ここでリターン.
        ---------------------------------------- -}

	    HeapWord* result = NULL;
	    if (!isHumongous(word_size)) {
	      result = attempt_allocation(word_size, &gc_count_before);
	    } else {
	      result = attempt_allocation_humongous(word_size, &gc_count_before);
	    }
	    if (result != NULL) {
	      return result;
	    }
	
    {- -------------------------------------------
  (1.1) VM_G1CollectForAllocation で GC を実行する
        ---------------------------------------- -}

	    // Create the garbage collection operation...
	    VM_G1CollectForAllocation op(gc_count_before, word_size);
	    // ...and get the VM thread to execute it.
	    VMThread::execute(&op);
	
    {- -------------------------------------------
  (1.1) もし自分が要求を出した GC が実行され (See: prologue_succeeded), 
        かつそれが GC_locker によって中断されなかった場合(See: pause_succeeded)は, 
        確保が成功したかどうかに関わらず, 結果をリターンする.
  
        (なお, pause_succeeded は GC_locker によって中断されたかどうかを示す.
         (See: G1CollectedHeap::do_collection_pause_at_safepoint()))
             
        (なお, 確保が成功し, かつサイズが大きすぎない場合 (isHumongous() ではない場合) には, 
        ここで barrier set の dirtying を行っている.)
        ---------------------------------------- -}

	    if (op.prologue_succeeded() && op.pause_succeeded()) {
	      // If the operation was successful we'll return the result even
	      // if it is NULL. If the allocation attempt failed immediately
	      // after a Full GC, it's unlikely we'll be able to allocate now.
	      HeapWord* result = op.result();
	      if (result != NULL && !isHumongous(word_size)) {
	        // Allocations that take place on VM operations do not do any
	        // card dirtying and we have to do it here. We only have to do
	        // this for non-humongous allocations, though.
	        dirty_young_block(result, word_size);
	      }
	      return result;

    {- -------------------------------------------
  (1.1) もし自分が要求を出した GC が実行されなかったか, 
        GC_locker によって中断されていた場合には, 
        このままループの次の周回にフォールスルー.
        ---------------------------------------- -}

	    } else {
	      assert(op.result() == NULL,
	             "the result should be NULL if the VM op did not succeed");
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // Give a warning if we seem to be looping forever.
	    if ((QueuedAllocationWarningCount > 0) &&
	        (try_count % QueuedAllocationWarningCount == 0)) {
	      warning("G1CollectedHeap::mem_allocate retries %d times", try_count);
	    }
	  }
	
  {- -------------------------------------------
  (1) (以下は決して到達しないパス)
      ---------------------------------------- -}

	  ShouldNotReachHere();
	  return NULL;
	}
	
```


