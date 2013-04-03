---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/collectorPolicy.cpp

### 名前(function name)
```
HeapWord* GenCollectorPolicy::mem_allocate_work(size_t size,
                                        bool is_tlab,
                                        bool* gc_overhead_limit_was_exceeded) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  GenCollectedHeap *gch = GenCollectedHeap::heap();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  debug_only(gch->check_for_valid_allocation_state());
	  assert(gch->no_gc_in_progress(), "Allocation during gc not allowed");
	
  {- -------------------------------------------
  (1) gc_overhead_limit_was_exceeded を初期化する
      (See above)
      ---------------------------------------- -}

	  // In general gc_overhead_limit_was_exceeded should be false so
	  // set it so here and reset it to true only if the gc time
	  // limit is being exceeded as checked below.
	  *gc_overhead_limit_was_exceeded = false;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  HeapWord* result = NULL;
	
  {- -------------------------------------------
  (1) 以下, メモリ確保が成功するかあるいは成功しないと判断するまで, 以下のループで処理を続ける
      (以下のループを抜けるのは return か throw した時だけ)
      (詳細は以下参照)
      ---------------------------------------- -}

	  // Loop until the allocation is satisified,
	  // or unsatisfied after GC.
	  for (int try_count = 1; /* return or throw */; try_count += 1) {
	    HandleMark hm; // discard any handles allocated in each iteration
	
    {- -------------------------------------------
  (1.1) もし Young Generation (以下の gen0) の Generation::should_allocate() が true を返したら, 
        まず Generation::par_allocate() で Young Generation からの確保を試みる.
      
        確保が成功したら, ここでリターン.
  
          (なお, Generation::should_allocate() は DefNewGeneration はオーバーライドしている模様. 
           他はデフォルトの Generation::should_allocate() が使われる模様.
           See: DefNewGeneration::should_allocate())
          (また, Generation::par_allocate() 自体は abstract なメソッドで, 
          実際はサブクラスがオーバーライドしたメソッドが呼び出される)
        ---------------------------------------- -}

	    // First allocation attempt is lock-free.
	    Generation *gen0 = gch->get_gen(0);
	    assert(gen0->supports_inline_contig_alloc(),
	      "Otherwise, must do alloc within heap lock");
	    if (gen0->should_allocate(size, is_tlab)) {
	      result = gen0->par_allocate(size, is_tlab);
	      if (result != NULL) {
	        assert(gch->is_in_reserved(result), "result not in heap");
	        return result;
	      }
	    }
	    unsigned int gc_count_before;  // read inside the Heap_lock locked region

    {- -------------------------------------------
  (1.1) (以下, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    {
	      MutexLocker ml(Heap_lock);

      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	      if (PrintGC && Verbose) {
	        gclog_or_tty->print_cr("TwoGenerationCollectorPolicy::mem_allocate_work:"
	                      " attempting locked slow path allocation");
	      }

      {- -------------------------------------------
  (1.1.1) GenCollectedHeap::attempt_allocation() での確保を試みる.
          確保が成功したら, ここでリターン.
          
          (なお first_only は, Young Generation だけで確保を試みるか, Old Generation まで見にいくか, を指定する.
          大きいオブジェクトであれば old 世代も見にいく.)
          ---------------------------------------- -}

	      // Note that only large objects get a shot at being
	      // allocated in later generations.
	      bool first_only = ! should_try_older_generation_allocation(size);
	
	      result = gch->attempt_allocation(size, is_tlab, first_only);
	      if (result != NULL) {
	        assert(gch->is_in_reserved(result), "result not in heap");
	        return result;
	      }
	
    {- -------------------------------------------
  (1.1) もし, 現在 GC が禁止されており(= GC_locker が active で), かつ 
        既に誰かが GC のエントリ処理を実行して GC の必要性を GC_locker に通知している場合, 
        以下の処理を行う.
        * もし TLAB の確保処理であれば, 
          確保は失敗とする (NULL をリターン)
        * もし...であれば(#TODO), 
          GenCollectorPolicy::expand_heap_and_allocate() でヒープを拡張してメモリを確保してみる.
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

	        if (is_tlab) {
	          return NULL;  // Caller will retry allocating individual object
	        }

      {- -------------------------------------------
  (1.1.1) もし...であれば(#TODO), GenCollectorPolicy::expand_heap_and_allocate() でヒープを拡張してメモリを確保してみる.
          確保できれば, ここでリターン.
          ---------------------------------------- -}

	        if (!gch->is_maximal_no_gc()) {
	          // Try and expand heap to satisfy request
	          result = expand_heap_and_allocate(size, is_tlab);
	          // result could be null if we are out of space
	          if (result != NULL) {
	            return result;
	          }
	        }
	
      {- -------------------------------------------
  (1.1.1) もしカレントスレッドが JNI のクリティカルセクションにいるのでなければ, 
          GC_locker のロックが解けて GC が実行可能になるまでここで待たせる.
            (なお, すでに GC の必要性は GC_locker に通知されているので(= GC_locker::is_active_and_needs_gc() が true なので), 
             待機が解けた時点では, 最後に critical section を抜けたスレッドが既に GC を呼び出している.
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
	          // Wait for JNI critical section to be exited
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
  (1.1) もし上の条件が成り立たなければ (= GC_locker::is_active_and_needs_gc() が false ならば), 
        GC 処理に備えて現在の total_collections() の値を取得し, 
        (See: total_collections())
        そのまま以下の GC 処理へと進む.
        ---------------------------------------- -}

	      // Read the gc count while the heap lock is held.
	      gc_count_before = Universe::heap()->total_collections();

    {- -------------------------------------------
  (1.1) (ここまでが, Heap_lock を取得した状態での排他処理)
        ---------------------------------------- -}

	    }
	
  {- -------------------------------------------
  (1) VM_GenCollectForAllocation で GC を実行する
      ---------------------------------------- -}

	    VM_GenCollectForAllocation op(size,
	                                  is_tlab,
	                                  gc_count_before);
	    VMThread::execute(&op);

  {- -------------------------------------------
  (1) もし GC が実行された場合は (= 同時に複数スレッドから GC 要求が来た場合には
      1つしか実行されないので, 自分が要求した GC が実行されたかどうかを調べている.
      See: prologue_succeeded), 結果に応じて以下の処理を行う.
      * もし GC_locker によって中断されていた場合には, ループの先頭に戻ってやり直す
      * GC 時間が制限を越えていれば, NULL を返す
      * それ以外なら, 確保結果をリターンする  (<= 確保失敗時には NULL ?? #TODO)
      ---------------------------------------- -}

	    if (op.prologue_succeeded()) {
	      result = op.result();

    {- -------------------------------------------
  (1.1) もし GC_locker によって中断されていた場合には, ループの先頭に戻ってやり直す.
        ---------------------------------------- -}

	      if (op.gc_locked()) {
	         assert(result == NULL, "must be NULL if gc_locked() is true");
	         continue;  // retry and/or stall as necessary
	      }
	
    {- -------------------------------------------
  (1.1) もし, GC 時間が制限を越えていれば (= AdaptiveSizePolicy::gc_overhead_limit_exceeded() が true ならば), 
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

	      // Allocation has failed and a collection
	      // has been done.  If the gc time limit was exceeded the
	      // this time, return NULL so that an out-of-memory
	      // will be thrown.  Clear gc_overhead_limit_exceeded
	      // so that the overhead exceeded does not persist.
	
	      const bool limit_exceeded = size_policy()->gc_overhead_limit_exceeded();
	      const bool softrefs_clear = all_soft_refs_clear();
	      assert(!limit_exceeded || softrefs_clear, "Should have been cleared");
	      if (limit_exceeded && softrefs_clear) {
	        *gc_overhead_limit_was_exceeded = true;
	        size_policy()->set_gc_overhead_limit_exceeded(false);
	        if (op.result() != NULL) {
	          CollectedHeap::fill_with_object(op.result(), size);
	        }
	        return NULL;
	      }

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	      assert(result == NULL || gch->is_in_reserved(result),
	             "result not in heap");

    {- -------------------------------------------
  (1.1) 確保結果をリターンする
        ---------------------------------------- -}

	      return result;
	    }
	
    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // Give a warning if we seem to be looping forever.
	    if ((QueuedAllocationWarningCount > 0) &&
	        (try_count % QueuedAllocationWarningCount == 0)) {
	          warning("TwoGenerationCollectorPolicy::mem_allocate_work retries %d times \n\t"
	                  " size=%d %s", try_count, size, is_tlab ? "(TLAB)" : "");
	    }
	  }
	}
	
```


