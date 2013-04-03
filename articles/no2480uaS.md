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
// There are two levels of allocation policy here.
//
// When an allocation request fails, the requesting thread must invoke a VM
// operation, transfer control to the VM thread, and await the results of a
// garbage collection. That is quite expensive, and we should avoid doing it
// multiple times if possible.
//
// To accomplish this, we have a basic allocation policy, and also a
// failed allocation policy.
//
// The basic allocation policy controls how you allocate memory without
// attempting garbage collection. It is okay to grab locks and
// expand the heap, if that can be done without coming to a safepoint.
// It is likely that the basic allocation policy will not be very
// aggressive.
//
// The failed allocation policy is invoked from the VM thread after
// the basic allocation policy is unable to satisfy a mem_allocate
// request. This policy needs to cover the entire range of collection,
// heap expansion, and out-of-memory conditions. It should make every
// attempt to allocate the requested memory.

// Basic allocation policy. Should never be called at a safepoint, or
// from the VM thread.
//
// This method must handle cases where many mem_allocate requests fail
// simultaneously. When that happens, only one VM operation will succeed,
// and the rest will not be executed. For that reason, this method loops
// during failed allocation attempts. If the java heap becomes exhausted,
// we rely on the size_policy object to force a bail out.
```

### 名前(function name)
```
HeapWord* ParallelScavengeHeap::mem_allocate(
                                     size_t size,
                                     bool is_noref,
                                     bool is_tlab,
                                     bool* gc_overhead_limit_was_exceeded) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!SafepointSynchronize::is_at_safepoint(), "should not be at safepoint");
	  assert(Thread::current() != (Thread*)VMThread::vm_thread(), "should not be in vm thread");
	  assert(!Heap_lock->owned_by_self(), "this thread should not own the Heap_lock");
	
  {- -------------------------------------------
  (1) gc_overhead_limit_was_exceeded を初期化する
      ---------------------------------------- -}

	  // In general gc_overhead_limit_was_exceeded should be false so
	  // set it so here and reset it to true only if the gc time
	  // limit is being exceeded as checked below.
	  *gc_overhead_limit_was_exceeded = false;
	
  {- -------------------------------------------
  (1) PSYoungGen::allocate() での確保を試みる.
      ---------------------------------------- -}

	  HeapWord* result = young_gen()->allocate(size, is_tlab);
	
  {- -------------------------------------------
  (1) もし, PSYoungGen::allocate() での確保が成功しなければ, 以下のループ内で確保を試みる.
      (以下, メモリ確保が成功するかあるいは成功しないと判断するまで, 以下のループで処理を続ける)
      ---------------------------------------- -}

	  uint loop_count = 0;
	  uint gc_count = 0;
	
	  while (result == NULL) {

    {- -------------------------------------------
  (1.1) (以下のコメントによると, GC が多発しないように total_collections() という値で制御している模様. 未読 #TODO)
        ---------------------------------------- -}

	    // We don't want to have multiple collections for a single filled generation.
	    // To prevent this, each thread tracks the total_collections() value, and if
	    // the count has changed, does not do a new collection.
	    //
	    // The collection count must be read only while holding the heap lock. VM
	    // operations also hold the heap lock during collections. There is a lock
	    // contention case where thread A blocks waiting on the Heap_lock, while
	    // thread B is holding it doing a collection. When thread A gets the lock,
	    // the collection count has already changed. To prevent duplicate collections,
	    // The policy MUST attempt allocations during the same period it reads the
	    // total_collections() value!

    {- -------------------------------------------
  (1.1) (以下, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    {
	      MutexLocker ml(Heap_lock);

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	      gc_count = Universe::heap()->total_collections();
	
    {- -------------------------------------------
  (1.1) PSYoungGen::allocate() での確保を試みる.
        確保が成功すれば, ここでリターン.
        ---------------------------------------- -}

	      result = young_gen()->allocate(size, is_tlab);
	
	      // (1) If the requested object is too large to easily fit in the
	      //     young_gen, or
	      // (2) If GC is locked out via GCLocker, young gen is full and
	      //     the need for a GC already signalled to GCLocker (done
	      //     at a safepoint),
	      // ... then, rather than force a safepoint and (a potentially futile)
	      // collection (attempt) for each allocation, try allocation directly
	      // in old_gen. For case (2) above, we may in the future allow
	      // TLAB allocation directly in the old gen.
	      if (result != NULL) {
	        return result;
	      }

    {- -------------------------------------------
  (1.1) もし, 今回の確保処理が TLAB の確保ではなく, かつ
        Young Gen にいれるには大きすぎる確保要求であれば (具体的には Eden の半分以上であれば), 
        PSOldGen::allocate() での確保を試みる.
        (この条件に付いては上のコメント("// (1) ...")も参照)
        ---------------------------------------- -}

	      if (!is_tlab &&
	          size >= (young_gen()->eden_space()->capacity_in_words(Thread::current()) / 2)) {
	        result = old_gen()->allocate(size, is_tlab);
	        if (result != NULL) {
	          return result;
	        }
	      }

    {- -------------------------------------------
  (1.1) もし, 現在 GC が禁止されており(= GC_locker が active で), かつ 
        既に誰かが GC のエントリ処理を実行して GC の必要性を GC_locker に通知している場合, 
        以下の処理を行う.
        * もし TLAB の確保処理であれば, 
          確保は失敗とする (NULL をリターン)
        * もしカレントスレッドが JNI のクリティカルセクションにいれば
          ここでリターン
        * もしカレントスレッドが JNI のクリティカルセクションにいるのでなければ, 
          GC_locker のロックが解けるまで待機し, その後ループの先頭に戻って確保処理をやり直す.
        ---------------------------------------- -}

	      if (GC_locker::is_active_and_needs_gc()) {

      {- -------------------------------------------
  (1.1.1) もし TLAB の確保処理であれば, ここでリターン(NULL を返す).
          (TLAB の確保なら, 失敗しても呼び出し元が
          実際に確保したいオブジェクトの確保処理をもう一度呼び出すだろう, と想定)
          ---------------------------------------- -}

	        // GC is locked out. If this is a TLAB allocation,
	        // return NULL; the requestor will retry allocation
	        // of an idividual object at a time.
	        if (is_tlab) {
	          return NULL;
	        }
	
      {- -------------------------------------------
  (1.1.1) もしカレントスレッドが JNI のクリティカルセクションにいるのでなければ, GC_locker のロックが解けて GC が実行可能になるまでここで待たせる.
            (なお, すでに GC の必要性は GC_locker に通知されているので(= GC_locker::is_active_and_needs_gc() が true なので), 待機が解けた時点では, 最後に critical section を抜けたスレッドが既に GC を呼び出している.
            なので, 待機が解けてもここでは GC はせず, ループの先頭に戻ってやり直すだけにしている)
          逆に, もしカレントスレッドが JNI のクリティカルセクションにいたら, ここでリターン (NULL をリターンする)
            (ただし, CheckJNICalls オプションが指定されている場合, fatal() で異常終了する)
          ---------------------------------------- -}

	        // If this thread is not in a jni critical section, we stall
	        // the requestor until the critical section has cleared and
	        // GC allowed. When the critical section clears, a GC is
	        // initiated by the last thread exiting the critical section; so
	        // we retry the allocation sequence from the beginning of the loop,
	        // rather than causing more, now probably unnecessary, GC attempts.
	        JavaThread* jthr = JavaThread::current();
	        if (!jthr->in_critical()) {
	          MutexUnlocker mul(Heap_lock);
	          GC_locker::stall_until_clear();
	          continue;
	        } else {
	          if (CheckJNICalls) {
	            fatal("Possible deadlock due to allocating while"
	                  " in jni critical section");
	          }
	          return NULL;
	        }
	      }

    {- -------------------------------------------
  (1.1) (ここまでが, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    }
	
  {- -------------------------------------------
  (1) もしここまでで確保に成功していなければ, VM_ParallelGCFailedAllocation で GC を実行する (詳細は以下参照).
      ---------------------------------------- -}

	    if (result == NULL) {
	
	      // Generate a VM operation
	      VM_ParallelGCFailedAllocation op(size, is_tlab, gc_count);
	      VMThread::execute(&op);
	
    {- -------------------------------------------
  (1.1) もし GC が実行された場合は (= 同時に複数スレッドから GC 要求が来た場合には
        1つしか実行されないので, 自分が要求した GC が実行されたかどうかを調べている.
        See: prologue_succeeded), 結果に応じて以下の処理を行う.
        * もし GC_locker によって中断されていた場合には, ループの先頭に戻ってやり直す
        * GC 時間が制限を越えていれば, NULL を返す
        * それ以外なら, 確保結果をリターンする  (<= 確保失敗時には NULL ?? #TODO)
        ---------------------------------------- -}

	      // Did the VM operation execute? If so, return the result directly.
	      // This prevents us from looping until time out on requests that can
	      // not be satisfied.
	      if (op.prologue_succeeded()) {
	        assert(Universe::heap()->is_in_or_null(op.result()),
	          "result not in heap");
	
      {- -------------------------------------------
  (1.1.1) もし GC_locker によって中断されていた場合には, ループの先頭に戻ってやり直す.
          ---------------------------------------- -}

	        // If GC was locked out during VM operation then retry allocation
	        // and/or stall as necessary.
	        if (op.gc_locked()) {
	          assert(op.result() == NULL, "must be NULL if gc_locked() is true");
	          continue;  // retry and/or stall as necessary
	        }
	
      {- -------------------------------------------
  (1.1.1) もし, GC 時間が制限を越えていれば (= AdaptiveSizePolicy::gc_overhead_limit_exceeded() が true ならば), 
          ここでリターン 
          (返値としては NULL をリターン. 呼び出し元が OutOfMemoryError を出してくれると想定)
  
          (なお正確に言うと, 実際のリターン条件は以下の2つの両方が成り立つこと.
             * GC 時間が制限を越えている
             * GC 時に全ての soft reference を消去済み 
               (= CollectorPolicy::all_soft_refs_clear() が true (See: ClearedAllSoftRefs))
           ただし if 文直前の assert によれば「GC 時間が制限を越えている場合は, 全ての soft reference は消去済み」なので, 
           実質的には「GC 時間が制限を越えていれば」リターンする模様.
           (この assert は正しい? 制限時間がものすごく短くても soft reference は消去される? #TODO))
  
          (なお, リターン前に, 引数の gc_overhead_limit_was_exceeded を true に設定している)
          (また, リターン前に gc_overhead_limit_exceeded の値は false に戻している. 
           これは, OutOfMemoryError をアプリケーションが handling して実行を継続することにした場合に, 
           次回の GC 以降に影響が残らないようにするため)
          (また, オブジェクト確保が成功してしまっていたら, CollectedHeap::fill_with_object() でその領域を dummy の配列で埋めている)
          ---------------------------------------- -}

	        // Exit the loop if the gc time limit has been exceeded.
	        // The allocation must have failed above ("result" guarding
	        // this path is NULL) and the most recent collection has exceeded the
	        // gc overhead limit (although enough may have been collected to
	        // satisfy the allocation).  Exit the loop so that an out-of-memory
	        // will be thrown (return a NULL ignoring the contents of
	        // op.result()),
	        // but clear gc_overhead_limit_exceeded so that the next collection
	        // starts with a clean slate (i.e., forgets about previous overhead
	        // excesses).  Fill op.result() with a filler object so that the
	        // heap remains parsable.
	        const bool limit_exceeded = size_policy()->gc_overhead_limit_exceeded();
	        const bool softrefs_clear = collector_policy()->all_soft_refs_clear();
	        assert(!limit_exceeded || softrefs_clear, "Should have been cleared");
	        if (limit_exceeded && softrefs_clear) {
	          *gc_overhead_limit_was_exceeded = true;
	          size_policy()->set_gc_overhead_limit_exceeded(false);

        {- -------------------------------------------
  (1.1.1.1) (トレース出力)
            ---------------------------------------- -}

	          if (PrintGCDetails && Verbose) {
	            gclog_or_tty->print_cr("ParallelScavengeHeap::mem_allocate: "
	              "return NULL because gc_overhead_limit_exceeded is set");
	          }
	          if (op.result() != NULL) {
	            CollectedHeap::fill_with_object(op.result(), size);
	          }
	          return NULL;
	        }
	
      {- -------------------------------------------
  (1.1.1) 確保結果をリターンする
          ---------------------------------------- -}

	        return op.result();
	      }
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // The policy object will prevent us from looping forever. If the
	    // time spent in gc crosses a threshold, we will bail out.
	    loop_count++;
	    if ((result == NULL) && (QueuedAllocationWarningCount > 0) &&
	        (loop_count % QueuedAllocationWarningCount == 0)) {
	      warning("ParallelScavengeHeap::mem_allocate retries %d times \n\t"
	              " size=%d %s", loop_count, size, is_tlab ? "(TLAB)" : "");
	    }
	  }
	
  {- -------------------------------------------
  (1) 結果をリターンする
      ---------------------------------------- -}

	  return result;
	}
	
```


