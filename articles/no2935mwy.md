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
  void work(int worker_i) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(Thread::current()->is_ConcurrentGC_thread(),
	           "this should only be done by a conc GC thread");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    ResourceMark rm;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (処理の開始時間を記録しておく)
      ---------------------------------------- -}

	    double start_vtime = os::elapsedVTime();
	
  {- -------------------------------------------
  (1) ConcurrentGCThread::stsJoin() を呼んで, 
      カレントスレッドを SuspendibleThreadSet による制御対象に登録しておく.
      (以降の処理は, 適宜 SuspendibleThreadSet::yield() しながら実行するため)
      ---------------------------------------- -}

	    ConcurrentGCThread::stsJoin();
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert((size_t) worker_i < _cm->active_tasks(), "invariant");

  {- -------------------------------------------
  (1) ConcurrentMark::task() を呼んで, 自分が担当する CMTask を取得する.
      ---------------------------------------- -}

	    CMTask* the_task = _cm->task(worker_i);

  {- -------------------------------------------
  (1) (以降の if ブロック内で Concurrent marking 処理を行う.
     
       なお, 処理の前後で CMTask::record_start_time() と CMTask::record_end_time() を呼び出して
       処理に掛かった時間を _elapsed_time_ms フィールドに記録しているが, 
       これはトレース出力用の処理 (現在は CMTask::print_stats() 内でのみ参照されている))
      ---------------------------------------- -}

	    the_task->record_start_time();

    {- -------------------------------------------
  (1.1) (既に処理が終了していたら (= CMTask::has_aborted() が false の場合), 以降のブロックの処理は省略)
        (これはどういうケース? #TODO)
        ---------------------------------------- -}

	    if (!_cm->has_aborted()) {

    {- -------------------------------------------
  (1.1) 以降の do...while ブロック内で, 以下のどちらかが成り立つまで CMTask::do_marking_step() を呼び続け, 
        Concurrent marking 処理を行う.
  
        * 並行して Full GC が発生してしまった場合 (= ConcurrentMark::has_aborted() が true の場合)
          (この場合は Concurrent marking を続けても意味が無いので終了する)
        * 途中で処理が中断されず, 出来る仕事は全てやりきった場合 (= CMTask::has_aborted() が false の場合)
          (この場合は Concurrent marking が完了したので終了する)
        ---------------------------------------- -}

	      do {

      {- -------------------------------------------
  (1.1.1) (変数宣言など)
          ---------------------------------------- -}

	        double start_vtime_sec = os::elapsedVTime();
	        double start_time_sec = os::elapsedTime();
	        double mark_step_duration_ms = G1ConcMarkStepDurationMillis;
	
      {- -------------------------------------------
  (1.1.1) CMTask::do_marking_step() を呼んで, Concurrent marking 処理を行う.
          ---------------------------------------- -}

	        the_task->do_marking_step(mark_step_duration_ms,
	                                  true /* do_stealing    */,
	                                  true /* do_termination */);
	
      {- -------------------------------------------
  (1.1.1) (変数宣言など)
          ---------------------------------------- -}

	        double end_time_sec = os::elapsedTime();
	        double end_vtime_sec = os::elapsedVTime();
	        double elapsed_vtime_sec = end_vtime_sec - start_vtime_sec;
	        double elapsed_time_sec = end_time_sec - start_time_sec;

      {- -------------------------------------------
  (1.1.1) 
          ---------------------------------------- -}

	        _cm->clear_has_overflown();
	
      {- -------------------------------------------
  (1.1.1) ConcurrentMark::do_yield_check() を呼んでおく.
  
          (この中では, 登録している SuspendibleThreadSet に対して 
           SuspendibleThreadSet::suspend_all() が呼ばれていれば, 待機処理を行う.
           この待機は SuspendibleThreadSet::resume_all() が呼ばれると解ける)
          ---------------------------------------- -}

	        bool ret = _cm->do_yield_check(worker_i);
	
      {- -------------------------------------------
  (1.1.1) まだループの終了条件が満たされていなければ (= 次のループも行う予定であれば), 
          Concurrent marking によるオーバーヘッドを平準化するため, 
          次のループを開始する前に適当な時間だけ os::sleep() で待機しておく
          (待機時間は今回の処理に掛かった時間(elapsed_vtime_sec)に比例).
  
          (なお, 待機している間だけはカレントスレッドを SuspendibleThreadSet による制御対象から削除しておく)
          ---------------------------------------- -}

	        jlong sleep_time_ms;
	        if (!_cm->has_aborted() && the_task->has_aborted()) {
	          sleep_time_ms =
	            (jlong) (elapsed_vtime_sec * _cm->sleep_factor() * 1000.0);
	          ConcurrentGCThread::stsLeave();
	          os::sleep(Thread::current(), sleep_time_ms, false);
	          ConcurrentGCThread::stsJoin();
	        }

      {- -------------------------------------------
  (1.1.1) (トレース出力)(#if 0 で消されているが...??)
          ---------------------------------------- -}

	        double end_time2_sec = os::elapsedTime();
	        double elapsed_time2_sec = end_time2_sec - start_time_sec;
	
	#if 0
	          gclog_or_tty->print_cr("CM: elapsed %1.4lf ms, sleep %1.4lf ms, "
	                                 "overhead %1.4lf",
	                                 elapsed_vtime_sec * 1000.0, (double) sleep_time_ms,
	                                 the_task->conc_overhead(os::elapsedTime()) * 8.0);
	          gclog_or_tty->print_cr("elapsed time %1.4lf ms, time 2: %1.4lf ms",
	                                 elapsed_time_sec * 1000.0, elapsed_time2_sec * 1000.0);
	#endif
	      } while (!_cm->has_aborted() && the_task->has_aborted());
	    }
	    the_task->record_end_time();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    guarantee(!the_task->has_aborted() || _cm->has_aborted(), "invariant");
	
  {- -------------------------------------------
  (1) 処理が終わったので, ConcurrentGCThread::stsLeave() を呼んで, 
      カレントスレッドを SuspendibleThreadSet による制御対象から削除する.
      ---------------------------------------- -}

	    ConcurrentGCThread::stsLeave();
	
  {- -------------------------------------------
  (1) (トレース出力用の処理)
  
      (ConcurrentMark::update_accum_task_vtime() を呼んで
       処理に掛かった時間を記録している.
       この結果は ConcurrentMark::_accum_task_vtime フィールドに格納され,
       現在は以下のパスでのみ参照されている.
       
       before_exit()
       -> G1CollectedHeap::print_tracing_info()
          -> ConcurrentMark::print_summary_info() (G1SummarizeConcMark オプション指定時のみ)
             -> ConcurrentMarkThread::vtime_accum() または ConcurrentMarkThread::vtime_mark_accum()
                -> ConcurrentMark::all_task_accum_vtime())
      ---------------------------------------- -}

	    double end_vtime = os::elapsedVTime();
	    _cm->update_accum_task_vtime(worker_i, end_vtime - start_vtime);
	  }
	
```


