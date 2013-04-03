---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.cpp

### 名前(function name)
```
void ConcurrentG1RefineThread::run() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ConcurrentGCThread::initialize_in_thread() を呼んで, 初期化を行う.
      ---------------------------------------- -}

	  initialize_in_thread();

  {- -------------------------------------------
  (1) ConcurrentGCThread::wait_for_universe_init() を呼んで, 
      HotSpot の初期化が終わるまで(= is_init_completed() が true になるまで)待機する.
      ---------------------------------------- -}

	  wait_for_universe_init();
	
  {- -------------------------------------------
  (1) もし _worker_id が ConcurrentG1Refine::worker_thread_num() 以上の場合は, 
      card を処理する代わりに, 
      ConcurrentG1RefineThread::run_young_rs_sampling() を呼んで
      HotSpot が終了するまで統計情報の更新処理を行う.
  
      (なお, ConcurrentG1RefineThread::run_young_rs_sampling() は, 
       HotSpot が終了するまで(= _should_terminate フィールドが true になるまで) 無限ループし, 
       延々と統計情報の更新処理を行う.
       See: ConcurrentG1RefineThread::stop())
    
      (その後, ConcurrentG1RefineThread::run_young_rs_sampling() が終了したら, 
       ConcurrentGCThread::terminate() を呼び出して, 
       ConcurrentG1RefineThread::stop() 内でこのスレッドの終了を待っているスレッドを起床させる
       (See: ConcurrentG1RefineThread::stop()))
      (その後, リターン)
      ---------------------------------------- -}

	  if (_worker_id >= cg1r()->worker_thread_num()) {
	    run_young_rs_sampling();
	    terminate();
	    return;
	  }
	
  {- -------------------------------------------
  (1) フィールドの初期化
      (処理の開始時間を記録しておく)
      ---------------------------------------- -}

	  _vtime_start = os::elapsedVTime();

  {- -------------------------------------------
  (1) (以下が ConcurrentG1RefineThread の処理のメインループ.
       HotSpot が終了するまで(= _should_terminate フィールドが true になるまで) この while ループを実行.
       See: ConcurrentG1RefineThread::stop())
      ---------------------------------------- -}

	  while (!_should_terminate) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    DirtyCardQueueSet& dcqs = JavaThread::dirty_card_queue_set();
	
    {- -------------------------------------------
  (1.1) ConcurrentG1RefineThread::wait_for_completed_buffers() を呼んで, 
        _should_terminate が true になるか, ConcurrentG1RefineThread::is_active() が true になるまで待機する.
  
        (ConcurrentG1RefineThread::ConcurrentG1RefineThread() 内のコメントにある通り,
         各 ConcurrentG1RefineThread はそれぞれ個別の Monitor オブジェクトを保持している.
         (0 版目の ConcurrentG1RefineThread だけは, DirtyCardQ_CBL_mon を使用する)
  
         各 ConcurrentG1RefineThread の monitor は,
         処理が多くスレッド数が足りないと判断された場合に
         1つ番号の小さい ConcurrentG1RefineThread から notify される. (下記の do ループ内を参照)
         (See: ConcurrentG1RefineThread::ConcurrentG1RefineThread()))
        ---------------------------------------- -}

	    // Wait for work
	    wait_for_completed_buffers();
	
    {- -------------------------------------------
  (1.1) もし待っている間に _should_terminate フィールドが true になっていたら, 
        ここでメインループを抜ける.
        ---------------------------------------- -}

	    if (_should_terminate) {
	      break;
	    }
	
    {- -------------------------------------------
  (1.1) SuspendibleThreadSet::join() を呼び出して, カレントスレッドを
        ConcurrentGCThread::_sts に格納されている SuspendibleThreadSet オブジェクトに登録しておく.
        (以降の処理は適宜 SuspendibleThreadSet::yield() しながら実行するので)
        (See: SuspendibleThreadSet)
        ---------------------------------------- -}

	    _sts.join();
	
    {- -------------------------------------------
  (1.1) (以下の do...while ループ内で DirtyCardQueueSet の処理を行う.
  
         処理は, DirtyCardQueueSet::apply_closure_to_completed_buffer() を呼び出すことで行う.
         また, 処理する量に合わせて動的に ConcurrentG1RefineThread を増減させる処理もここで行っている.)
        ---------------------------------------- -}

	    do {

      {- -------------------------------------------
  (1.1.1) (変数宣言など)
          ---------------------------------------- -}

	      int curr_buffer_num = (int)dcqs.completed_buffers_num();

      {- -------------------------------------------
  (1.1.1) 
          (See: _yellow_zone, G1CollectorPolicy::adjust_concurrent_refinement(), PtrQueueSet::process_or_enqueue_complete_buffer())
          ---------------------------------------- -}

	      // If the number of the buffers falls down into the yellow zone,
	      // that means that the transition period after the evacuation pause has ended.
	      if (dcqs.completed_queue_padding() > 0 && curr_buffer_num <= cg1r()->yellow_zone()) {
	        dcqs.set_completed_queue_padding(0);
	      }
	
      {- -------------------------------------------
  (1.1.1) もし処理する量が少なければ, ConcurrentG1RefineThread::deactivate() を呼んで
          is_active() が false を返すようにした後, break してループから抜ける.
          (次の ConcurrentG1RefineThread::wait_for_completed_buffers() で眠りにつくようになる)
          ---------------------------------------- -}

	      if (_worker_id > 0 && curr_buffer_num <= _deactivation_threshold) {
	        // If the number of the buffer has fallen below our threshold
	        // we should deactivate. The predecessor will reactivate this
	        // thread should the number of the buffers cross the threshold again.
	        deactivate();
	        break;
	      }
	
      {- -------------------------------------------
  (1.1.1) もし処理する量が多く, かつ1つ次の番号の ConcurrentG1RefineThread が稼働していなければ, 
          ConcurrentG1RefineThread::activate() を呼んで 
          1つ次の番号の ConcurrentG1RefineThread を稼働させる.
          ---------------------------------------- -}

	      // Check if we need to activate the next thread.
	      if (_next != NULL && !_next->is_active() && curr_buffer_num > _next->_threshold) {
	        _next->activate();
	      }

      {- -------------------------------------------
  (1.1.1) DirtyCardQueueSet::apply_closure_to_completed_buffer(int worker_i, int stop_at, bool during_pause) を呼んで
          ...#TODO
          ---------------------------------------- -}

	    } while (dcqs.apply_closure_to_completed_buffer(_worker_id + _worker_id_offset, cg1r()->green_zone()));
	
    {- -------------------------------------------
  (1.1) もしループを抜けたこの時点でも is_active() が true のままだったら, 
        ConcurrentG1RefineThread::deactivate() を呼んで false に変えておく.
        ---------------------------------------- -}

	    // We can exit the loop above while being active if there was a yield request.
	    if (is_active()) {
	      deactivate();
	    }
	
    {- -------------------------------------------
  (1.1) SuspendibleThreadSet::leave() を呼び出して
        ConcurrentGCThread::_sts に格納されている SuspendibleThreadSet オブジェクト内から
        カレントスレッドの登録を削除する
        (See: SuspendibleThreadSet)
        ---------------------------------------- -}

	    _sts.leave();
	
    {- -------------------------------------------
  (1.1) (トレース出力用の処理)
  
        (_vtime_accum は現在は
         ConcurrentMark::print_summary_info() と 
         PrintRSThreadVTimeClosure 内でのみ参照されている)
        ---------------------------------------- -}

	    if (os::supports_vtime()) {
	      _vtime_accum = (os::elapsedVTime() - _vtime_start);
	    } else {
	      _vtime_accum = 0.0;
	    }

  {- -------------------------------------------
  (1) (ここまでが ConcurrentG1RefineThread の処理のメインループ. _should_terminate が true の間は以上の処理を繰り返す)
      ---------------------------------------- -}

	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_should_terminate, "just checking");

  {- -------------------------------------------
  (1) ConcurrentGCThread::terminate() を呼び出して, 
      ConcurrentG1RefineThread::stop() 内でこのスレッドの終了を待っているスレッドを起床させる.
      (See: ConcurrentG1RefineThread::stop())
      ---------------------------------------- -}

	  terminate();
	}
	
```


