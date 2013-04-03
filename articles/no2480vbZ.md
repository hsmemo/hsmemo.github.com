---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vmThread.cpp

### 名前(function name)
```
void VMThread::loop() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_cur_vm_operation == NULL, "no current one should be executing");
	
  {- -------------------------------------------
  (1) (以下の while ループが VMThread のメイン処理.
       VMThread の lifetime のほとんどは, この無限ループに費やされる.
       VMThread::should_terminate() が true になったら, VMThread は無限ループを抜けて終了する.
       
       なお, VMThread::should_terminate() は HotSpot の終了処理が開始されると true になる.
       See: VMThread::wait_for_vm_thread_exit())
      ---------------------------------------- -}

	  while(true) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    VM_Operation* safepoint_ops = NULL;

    {- -------------------------------------------
  (1.1) 新しい VM Operation 要求 (VM_Operation オブジェクト) を取得する.
        (以下のブロック内で新しい VM Operation 要求が来るまで待機)
        ---------------------------------------- -}

	    //
	    // Wait for VM operation
	    //
	    // use no_safepoint_check to get lock without attempting to "sneak"
	    { MutexLockerEx mu_queue(VMOperationQueue_lock,
	                             Mutex::_no_safepoint_check_flag);
	
      {- -------------------------------------------
  (1.1.1) まず, VMOperationQueue::remove_next() で
          新しい VM_Operation オブジェクトが来ていないか確認する.
          ---------------------------------------- -}

	      // Look for new operation
	      assert(_cur_vm_operation == NULL, "no current one should be executing");
	      _cur_vm_operation = _vm_queue->remove_next();
	
      {- -------------------------------------------
  (1.1.1) (トレース出力)
          ---------------------------------------- -}

	      // Stall time tracking code
	      if (PrintVMQWaitTime && _cur_vm_operation != NULL &&
	          !_cur_vm_operation->evaluate_concurrently()) {
	        long stall = os::javaTimeMillis() - _cur_vm_operation->timestamp();
	        if (stall > 0)
	          tty->print_cr("%s stall: %Ld",  _cur_vm_operation->name(), stall);
	      }
	
      {- -------------------------------------------
  (1.1.1) もし VM_Operation オブジェクトが無ければ, 
          VMThread::should_terminate()が true になるか VM_Operation オブジェクトが得られるまで
          以下の while ループ内で待ち続ける.
          ---------------------------------------- -}

	      while (!should_terminate() && _cur_vm_operation == NULL) {

        {- -------------------------------------------
  (1.1.1.1) VMOperationQueue_lock に対する Monitor::wait() で VM_Operation オブジェクトが来るまで待つ.
            (ただし, 少なくともある程度の時間間隔で safepoint が行われるよう, 
             この wait() にはタイムアウト時間が指定されている.
             現在は GuaranteedSafepointInterval でタイムアウト.)
            ---------------------------------------- -}

	        // wait with a timeout to guarantee safepoints at regular intervals
	        bool timedout =
	          VMOperationQueue_lock->wait(Mutex::_no_safepoint_check_flag,
	                                      GuaranteedSafepointInterval);
	
        {- -------------------------------------------
  (1.1.1.1) (以下は wait() が解けた後の処理)
            ---------------------------------------- -}

        {- -------------------------------------------
  (1.1.1.1) (トラブルシューティング用の処理) (関連する product オプションが指定されている場合にのみ実行) (See: SelfDestructTimer)
            (もし起動後の経過時間が指定時間を超えていれば, ここで HotSpot を終了させる)
            (終了させる場合には, ついでに(トレース出力)も出している)
            ---------------------------------------- -}

	        // Support for self destruction
	        if ((SelfDestructTimer != 0) && !is_error_reported() &&
	            (os::elapsedTime() > SelfDestructTimer * 60)) {
	          tty->print_cr("VM self-destructed");
	          exit(-1);
	        }
	
        {- -------------------------------------------
  (1.1.1.1) もし wait() がタイムアウトで終了していた場合は, 
            SafepointSynchronize::is_cleanup_needed() を呼んで
            Safepoint 停止が必要かどうかを確認する.
    
            必要な場合には, ここで SafepointSynchronize::begin() と 
            SafepointSynchronize::end() を呼び出す.
    
            (これは, 少なくともある程度の時間間隔で safepoint を実行するための処理)
  
            (なお, デバッグ用の SafepointALot オプションが指定されている場合は, 
             SafepointSynchronize::is_cleanup_needed() の値に寄らず
             Safepoint 停止処理が行われる.)
            ---------------------------------------- -}

	        if (timedout && (SafepointALot ||
	                         SafepointSynchronize::is_cleanup_needed())) {
	          MutexUnlockerEx mul(VMOperationQueue_lock,
	                              Mutex::_no_safepoint_check_flag);
	          // Force a safepoint since we have not had one for at least
	          // 'GuaranteedSafepointInterval' milliseconds.  This will run all
	          // the clean-up processing that needs to be done regularly at a
	          // safepoint
	          SafepointSynchronize::begin();
	          #ifdef ASSERT
	            if (GCALotAtAllSafepoints) InterfaceSupport::check_gc_alot();
	          #endif
	          SafepointSynchronize::end();
	        }

        {- -------------------------------------------
  (1.1.1.1) VMOperationQueue::remove_next() で, 新しい VM_Operation オブジェクトを取得する.
            ---------------------------------------- -}

	        _cur_vm_operation = _vm_queue->remove_next();
	
        {- -------------------------------------------
  (1.1.1.1) 新しい VM_Operation オブジェクトを取得できた場合, 
            それが safepoint での実行を必要とするものかどうかを確認する.
            もしそうであれば, ついでに safepoint で実行しないといけない他の VM_Operation もまとめて取得しておく.
            ---------------------------------------- -}

	        // If we are at a safepoint we will evaluate all the operations that
	        // follow that also require a safepoint
	        if (_cur_vm_operation != NULL &&
	            _cur_vm_operation->evaluate_at_safepoint()) {
	          safepoint_ops = _vm_queue->drain_at_safepoint_priority();
	        }
	      }
	
      {- -------------------------------------------
  (1.1.1) なお, should_terminate() が true になったら, VMThread は無限ループを抜ける
          ---------------------------------------- -}

	      if (should_terminate()) break;
	    } // Release mu_queue_lock

    {- -------------------------------------------
  (1.1) (ここまでが, 新しい VM Operation 要求 (VM_Operation オブジェクト) を取得する処理)
        ---------------------------------------- -}
	
    {- -------------------------------------------
  (1.1) 次に, 取得した VM_Operation オブジェクトを実際に実行する.
        (以下のブロック内で VMThread::evaluate_operation() により VM_Operation を実行. 
         なお, safepoint 停止が必要かどうかに応じて処理を分けている. 
         詳細は以下参照)
        ---------------------------------------- -}

	    //
	    // Execute VM operation
	    //
	    { HandleMark hm(VMThread::vm_thread());
	
      {- -------------------------------------------
  (1.1.1) (トレース出力) (See: EventMark)
          ---------------------------------------- -}

	      EventMark em("Executing VM operation: %s", vm_operation()->name());

      {- -------------------------------------------
  (1.1.1) (assert)
          ---------------------------------------- -}

	      assert(_cur_vm_operation != NULL, "we should have found an operation to execute");
	
      {- -------------------------------------------
  (1.1.1) もし VMThreadHintNoPreempt オプションが指定されていれば, 
          VM_Operation 実行中に OS にコンテキストスイッチされないよう,
          OS に対して time slice を多めに与えてくれるように要望を出しておく.
          (See: VMThreadHintNoPreempt)
          ---------------------------------------- -}

	      // Give the VM thread an extra quantum.  Jobs tend to be bursty and this
	      // helps the VM thread to finish up the job.
	      // FIXME: When this is enabled and there are many threads, this can degrade
	      // performance significantly.
	      if( VMThreadHintNoPreempt )
	        os::hint_no_preempt();
	
      {- -------------------------------------------
  (1.1.1) (以下は, safepoint 停止が必要なケースの処理.
          SafepointSynchronize::begin() と SafepointSynchronize::end() で safepoint 停止させ,
          その中で VMThread::evaluate_operation() による VM_Operation の実行処理を行う.
    
          なお, この場合は, 上で書いたように safepoint を要求する VM_Operation を全部まとめて実行している.
          (safepoint 停止する回数を減らすため))
          ---------------------------------------- -}

	      // If we are at a safepoint we will evaluate all the operations that
	      // follow that also require a safepoint
	      if (_cur_vm_operation->evaluate_at_safepoint()) {
	
        {- -------------------------------------------
  (1.1.1.1) safepoint 停止するケースでは GC 処理が行われるかもしれないため, 
            実行予定の VM_Operation オブジェクトを _drain_list フィールドに登録しておく.
            ---------------------------------------- -}

	        _vm_queue->set_drain_list(safepoint_ops); // ensure ops can be scanned
	
        {- -------------------------------------------
  (1.1.1.1) SafepointSynchronize::begin() を呼び出す.
            (ここからが safepoint 内での処理)
            ---------------------------------------- -}

	        SafepointSynchronize::begin();

        {- -------------------------------------------
  (1.1.1.1) VMThread::evaluate_operation() で取得した VM_Operation を実行.
            ---------------------------------------- -}

	        evaluate_operation(_cur_vm_operation);

        {- -------------------------------------------
  (1.1.1.1) safepoint で実行しないといけない他の VM_Operation (= 上で取得した safepoint_ops の中身) も全部実行する.
            ---------------------------------------- -}

	        // now process all queued safepoint ops, iteratively draining
	        // the queue until there are none left
	        do {
	          _cur_vm_operation = safepoint_ops;
	          if (_cur_vm_operation != NULL) {
	            do {
	              // evaluate_operation deletes the op object so we have
	              // to grab the next op now
	              VM_Operation* next = _cur_vm_operation->next();
	              _vm_queue->set_drain_list(next);
	              evaluate_operation(_cur_vm_operation);
	              _cur_vm_operation = next;
	              if (PrintSafepointStatistics) {
	                SafepointSynchronize::inc_vmop_coalesced_count();
	              }
	            } while (_cur_vm_operation != NULL);
	          }

        {- -------------------------------------------
  (1.1.1.1) (なお, 上の do..while ループで一応全部実行したはずだが, 
            排他的に処理しているわけではない(ロックは取っていない)ので, 他のスレッドによって新しく追加された恐れはある.
            なので, safepoint 停止する回数を減らすために, 以下でもう一度チェックしている.)
            ---------------------------------------- -}

	          // There is a chance that a thread enqueued a safepoint op
	          // since we released the op-queue lock and initiated the safepoint.
	          // So we drain the queue again if there is anything there, as an
	          // optimization to try and reduce the number of safepoints.
	          // As the safepoint synchronizes us with JavaThreads we will see
	          // any enqueue made by a JavaThread, but the peek will not
	          // necessarily detect a concurrent enqueue by a GC thread, but
	          // that simply means the op will wait for the next major cycle of the
	          // VMThread - just as it would if the GC thread lost the race for
	          // the lock.
	          if (_vm_queue->peek_at_safepoint_priority()) {
	            // must hold lock while draining queue
	            MutexLockerEx mu_queue(VMOperationQueue_lock,
	                                     Mutex::_no_safepoint_check_flag);
	            safepoint_ops = _vm_queue->drain_at_safepoint_priority();
	          } else {
	            safepoint_ops = NULL;
	          }
	        } while(safepoint_ops != NULL);
	
        {- -------------------------------------------
  (1.1.1.1) safepoint 処理が終わったので, _drain_list フィールドを元に戻す.
            ---------------------------------------- -}

	        _vm_queue->set_drain_list(NULL);
	
        {- -------------------------------------------
  (1.1.1.1) 最後に SafepointSynchronize::end() を呼び出す.
            (ここまでが safepoint 内での処理)
            ---------------------------------------- -}

	        // Complete safepoint synchronization
	        SafepointSynchronize::end();
	
      {- -------------------------------------------
  (1.1.1) (ここまでが, Safepoint 停止が必要なケースの処理)
          ---------------------------------------- -}

      {- -------------------------------------------
  (1.1.1) (以下は, Safepoint 停止が不要なケースの処理.
           このまま VMThread::evaluate_operation() を呼び出す.
           
           なお, TraceLongCompiles オプションが指定されている場合には(トレース出力)処理も行っている)
          ---------------------------------------- -}

	      } else {  // not a safepoint operation
	        if (TraceLongCompiles) {
	          elapsedTimer t;
	          t.start();
	          evaluate_operation(_cur_vm_operation);
	          t.stop();
	          double secs = t.seconds();
	          if (secs * 1e3 > LongCompileThreshold) {
	            // XXX - _cur_vm_operation should not be accessed after
	            // the completed count has been incremented; the waiting
	            // thread may have already freed this memory.
	            tty->print_cr("vm %s: %3.7f secs]", _cur_vm_operation->name(), secs);
	          }
	        } else {
	          evaluate_operation(_cur_vm_operation);
	        }
	
        {- -------------------------------------------
  (1.1.1.1) _cur_vm_operation の値をリセットしておく
            ---------------------------------------- -}

	        _cur_vm_operation = NULL;
	      }
	    }

    {- -------------------------------------------
  (1.1) (ここまでが, 取得した VM_Operation オブジェクトを実際に実行する処理)
        ---------------------------------------- -}
	
    {- -------------------------------------------
  (1.1) VM_Operation の完了を待っている JavaThread がいるかもしれないので, notify_all() しておく.
        (See: VMThread::execute())
        ---------------------------------------- -}

	    //
	    //  Notify (potential) waiting Java thread(s) - lock without safepoint
	    //  check so that sneaking is not possible
	    { MutexLockerEx mu(VMOperationRequest_lock,
	                       Mutex::_no_safepoint_check_flag);
	      VMOperationRequest_lock->notify_all();
	    }
	
    {- -------------------------------------------
  (1.1) 最後に, 少なくともある程度の時間間隔で Safepoint が行われるよう, 
        SafepointSynchronize::is_cleanup_needed() を呼んで
        Safepoint 停止が必要かどうかを確認しておく.
    
        SafepointSynchronize::is_cleanup_needed() が true であり, かつ
        SafepointSynchronize::last_non_safepoint_interval() で取得した値が
        GuaranteedSafepointInterval オプションの値を超えていたら, 
        ここで SafepointSynchronize::begin() と SafepointSynchronize::end() を呼び出す.
        (ただし, GuaranteedSafepointInterval が 0 の場合は, この処理は行わない)
  
    
        (なお, デバッグ用の SafepointALot オプションが指定されている場合は, 
         SafepointSynchronize::is_cleanup_needed() さえ true であれば, 
         SafepointSynchronize::last_non_safepoint_interval() や GuaranteedSafepointInterval の値に寄らず
         Safepoint 停止処理が行われる.)
        ---------------------------------------- -}

	    //
	    // We want to make sure that we get to a safepoint regularly.
	    //
	    if (SafepointALot || SafepointSynchronize::is_cleanup_needed()) {
	      long interval          = SafepointSynchronize::last_non_safepoint_interval();
	      bool max_time_exceeded = GuaranteedSafepointInterval != 0 && (interval > GuaranteedSafepointInterval);
	      if (SafepointALot || max_time_exceeded) {
	        HandleMark hm(VMThread::vm_thread());
	        SafepointSynchronize::begin();
	        SafepointSynchronize::end();
	      }
	    }

  {- -------------------------------------------
  (1) (以上の処理を無限ループで繰り返す)
      ---------------------------------------- -}

	  }
	}
	
```


