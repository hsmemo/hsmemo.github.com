---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/memoryPool.cpp

### 名前(function name)
```
void MemoryPool::set_usage_sensor_obj(instanceHandle sh) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MemoryPool::set_sensor_obj_at() を呼び出して, 
      引数で渡されてきた sun.management.Sensor オブジェクトに
      対応づけた SensorInfo オブジェクトを作り, 
      _usage_sensor フィールドに設定する.
      ---------------------------------------- -}

	  set_sensor_obj_at(&_usage_sensor, sh);
	}
	
```


