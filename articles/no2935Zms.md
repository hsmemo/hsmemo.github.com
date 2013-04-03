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
  (1) (この処理はとりあえず全ての GangTask に割り振られるので, 自分には仕事はないかもしれない.
       まず自分に仕事があるかどうかを確認し, なければ以降の処理は省略する)
      ---------------------------------------- -}

	    // Since all available tasks are actually started, we should
	    // only proceed if we're supposed to be actived.
	    if ((size_t)worker_i < _cm->active_tasks()) {

  {- -------------------------------------------
  (1) ConcurrentMark::task() を呼んで, 自分が担当する CMTask を取得する.
      ---------------------------------------- -}

	      CMTask* task = _cm->task(worker_i);

  {- -------------------------------------------
  (1) 以下のどちらかが成り立つまで CMTask::do_marking_step() を呼び続け, Final marking 処理を行う.
  
      * 途中で処理が中断されず, 出来る仕事は全てやりきった場合 (= CMTask::has_aborted() が false の場合)
        (この場合は Final marking が完了したので終了する)
      * ConcurrentMark 内のスタックにこれ以上ものを詰めない場合 (= ConcurrentMark::has_overflown() が true の場合)
        (この場合は Final marking が失敗したので終了する)
    
      (なお, 処理の前後で CMTask::record_start_time() と CMTask::record_end_time() を呼び出して
       処理に掛かった時間を _elapsed_time_ms フィールドに記録しているが, 
       これはトレース出力用の処理 (現在は CMTask::print_stats() 内でのみ参照されている))
      ---------------------------------------- -}

	      task->record_start_time();
	      do {
	        task->do_marking_step(1000000000.0 /* something very large */,
	                              true /* do_stealing    */,
	                              true /* do_termination */);
	      } while (task->has_aborted() && !_cm->has_overflown());
	      // If we overflow, then we do not want to restart. We instead
	      // want to abort remark and do concurrent marking again.
	      task->record_end_time();
	    }
	  }
	
```


