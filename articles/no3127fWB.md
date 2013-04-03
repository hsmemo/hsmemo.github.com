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
void ConcurrentG1RefineThread::run_young_rs_sampling() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  DirtyCardQueueSet& dcqs = JavaThread::dirty_card_queue_set();

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _vtime_start = os::elapsedVTime();

  {- -------------------------------------------
  (1) (HotSpot が終了するまで(= _should_terminate フィールドが true になるまで) この while ループを実行.
       See: ConcurrentG1RefineThread::stop())
      ---------------------------------------- -}

	  while(!_should_terminate) {

    {- -------------------------------------------
  (1.1) 
        ---------------------------------------- -}

	    _sts.join();
	    sample_young_list_rs_lengths();
	    _sts.leave();
	
    {- -------------------------------------------
  (1.1) _vtime_accum フィールドの値を更新しておく
        ---------------------------------------- -}

	    if (os::supports_vtime()) {
	      _vtime_accum = (os::elapsedVTime() - _vtime_start);
	    } else {
	      _vtime_accum = 0.0;
	    }
	
    {- -------------------------------------------
  (1.1) もし HotSpot が終了していれば(= _should_terminate フィールドが true であれば) この while ループから抜ける.
        そうでなければ, Mutex::_no_safepoint_check_flag に対して Monitor::wait() を呼び出して待機する.
        ---------------------------------------- -}

	    MutexLockerEx x(_monitor, Mutex::_no_safepoint_check_flag);
	    if (_should_terminate) {
	      break;
	    }
	    _monitor->wait(Mutex::_no_safepoint_check_flag, G1ConcRefinementServiceIntervalMillis);
	  }
	}
	
```


