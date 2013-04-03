---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.cpp

### 名前(function name)
```
void LowMemoryDetector::process_sensor_changes(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	  HandleMark hm(THREAD);
	
  {- -------------------------------------------
  (1) 全ての MemoryPool を辿り, 
      usage sensor および gc usage sensor 内の未処理の通知に対する送信処理を行う
      (SensorInfo::has_pending_requests() で未処理の通知があるかどうかを確認し, 
       あれば SensorInfo::process_pending_requests() で送信処理を行う)
      ---------------------------------------- -}

	  // No need to hold Service_lock to call out to Java
	  int num_memory_pools = MemoryService::num_memory_pools();
	  for (int i = 0; i < num_memory_pools; i++) {
	    MemoryPool* pool = MemoryService::get_memory_pool(i);
	    SensorInfo* sensor = pool->usage_sensor();
	    SensorInfo* gc_sensor = pool->gc_usage_sensor();
	    if (sensor != NULL && sensor->has_pending_requests()) {
	      sensor->process_pending_requests(CHECK);
	    }
	    if (gc_sensor != NULL && gc_sensor->has_pending_requests()) {
	      gc_sensor->process_pending_requests(CHECK);
	    }
	  }
	}
	
```


