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
void LowMemoryDetector::detect_low_memory() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (Service_lock に対して notify_all() を呼び出すので, Service_lock のロックを取っておく)
      ---------------------------------------- -}

	  MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	
  {- -------------------------------------------
  (1) 全ての MemoryPool を辿り, 
      それらに対応づけられている usage sensor について
      もしそれが閾値を超過している可能性があれば, 
      SensorInfo::set_gauge_sensor_level() で
      通知の発生条件を満たしたかどうかを判定していく.
      ---------------------------------------- -}

	  bool has_pending_requests = false;
	  int num_memory_pools = MemoryService::num_memory_pools();
	  for (int i = 0; i < num_memory_pools; i++) {
	    MemoryPool* pool = MemoryService::get_memory_pool(i);
	    SensorInfo* sensor = pool->usage_sensor();
	    if (sensor != NULL &&
	        pool->usage_threshold()->is_high_threshold_supported() &&
	        pool->usage_threshold()->high_threshold() != 0) {
	      MemoryUsage usage = pool->get_memory_usage();
	      sensor->set_gauge_sensor_level(usage,
	                                     pool->usage_threshold());
	      has_pending_requests = has_pending_requests || sensor->has_pending_requests();
	    }
	  }
	
  {- -------------------------------------------
  (1) 上で調べた sensor の中に
      1つでも新しい通知が発生したものがあれば
      Service_lock に対して Monitor::notify_all() を呼び出して, ServiceThread を起床させる.
      ---------------------------------------- -}

	  if (has_pending_requests) {
	    Service_lock->notify_all();
	  }
	}
	
```


