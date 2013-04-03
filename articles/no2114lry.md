---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.cpp
### 説明(description)

```
// This method could be called from any Java threads
// and also VMThread.
```

### 名前(function name)
```
void LowMemoryDetector::detect_low_memory(MemoryPool* pool) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし, 
      * 引数で指定された MemoryPool にまだ usage sensor が作成されていなかったり, 
      * usage threshold が high threshold に対応していなかったり, 
      * high threshold の値が設定されていなかったり
      という場合には, 
      何も通知すべきことはないので, ここでリターン.
      ---------------------------------------- -}

	  SensorInfo* sensor = pool->usage_sensor();
	  if (sensor == NULL ||
	      !pool->usage_threshold()->is_high_threshold_supported() ||
	      pool->usage_threshold()->high_threshold() == 0) {
	    return;
	  }
	
	  {

  {- -------------------------------------------
  (1) (Service_lock に対して notify_all() を呼び出すので, Service_lock のロックを取っておく)
      ---------------------------------------- -}

	    MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	
  {- -------------------------------------------
  (1) SensorInfo::set_gauge_sensor_level() で, 
      新しい通知の発生条件を満たしたかどうかを判定する.
      ---------------------------------------- -}

	    MemoryUsage usage = pool->get_memory_usage();
	    sensor->set_gauge_sensor_level(usage,
	                                   pool->usage_threshold());

  {- -------------------------------------------
  (1) もし, 新しい通知が発生していれば, 
      Service_lock に対して Monitor::notify_all() を呼び出して, 
      ServiceThread を起床させる.
      ---------------------------------------- -}

	    if (sensor->has_pending_requests()) {
	      // notify sensor state update
	      Service_lock->notify_all();
	    }
	  }
	}
	
```


