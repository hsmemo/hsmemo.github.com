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
bool ConcurrentG1RefineThread::is_active() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  DirtyCardQueueSet& dcqs = JavaThread::dirty_card_queue_set();

  {- -------------------------------------------
  (1) worker_id の値に応じて, 以下のどちらかの値をリターンする.
      * worker_id が 0 番ではない場合:
        _active フィールドの値をリターンする.
      * worker_id が 0 番の場合:
        PtrQueueSet::process_completed_buffers() の値をリターンする
        (= DirtyCardQueueSet の _process_completed フィールドの値をリターンする).
  
      (See: ConcurrentG1RefineThread::activate(), ConcurrentG1RefineThread::deactivate())
      ---------------------------------------- -}

	  return _worker_id > 0 ? _active : dcqs.process_completed_buffers();
	}
	
```


