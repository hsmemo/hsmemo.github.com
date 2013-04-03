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
void SensorInfo::clear(int count, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.Sensor.clear() を呼び出す.
      ---------------------------------------- -}

	  if (_sensor_obj != NULL) {
	    klassOop k = Management::sun_management_Sensor_klass(CHECK);
	    instanceKlassHandle sensorKlass (THREAD, k);
	    Handle sensor(THREAD, _sensor_obj);
	
	    JavaValue result(T_VOID);
	    JavaCallArguments args(sensor);
	    args.push_int((int) count);
	    JavaCalls::call_virtual(&result,
	                            sensorKlass,
	                            vmSymbols::clear_name(),
	                            vmSymbols::int_void_signature(),
	                            &args,
	                            CHECK);
	  }
	
  {- -------------------------------------------
  (1) Sensor の状態を off に変更し, 送信した分だけカウンタを増減しておく.
      ---------------------------------------- -}

	  {
	    // Holds Service_lock and update the sensor state
	    MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	    _sensor_on = false;
	    _pending_clear_count = 0;
	    _pending_trigger_count = _pending_trigger_count - count;
	  }
	}
	
```


