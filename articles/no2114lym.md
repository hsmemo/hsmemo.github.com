---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.cpp
### 説明(description)
使用量が閾値付近で細かく振動した場合に
何度も通知が発生しないように, 
low threshold や high threshold をまたいだ時に出る通知(trigger or clear)は,
同じ種類のものは続けて出ないようになっている.
 
  * いったん high threshold を上回ると, sensor が trigger される (trigger 通知が生成される).
    実際に trigger 通知に送信されると, sensor は on 状態になる.
    trigger が一旦生成された後は, 次に clear が生成されるまでは, 
    high threshold を上回っても何も起きない.

  * 次に, trigger が生じて以降に, 使用量が low threshold を下回ると, 
    sensor は clear される (clear 通知が生成される).
    実際に clear 通知に送信されると, sensor は off 状態になる.
    clear が一旦生成された後は, 次に trigger が生成されるまで, 
    low threshold を下回っても何も起きない.

```
// When this method is used, the memory usage is monitored
// as a gauge attribute.  Sensor notifications (trigger or
// clear) is only emitted at the first time it crosses
// a threshold.
//
// High and low thresholds are designed to provide a
// hysteresis mechanism to avoid repeated triggering
// of notifications when the attribute value makes small oscillations
// around the high or low threshold value.
//
// The sensor will be triggered if:
//  (1) the usage is crossing above the high threshold and
//      the sensor is currently off and no pending
//      trigger requests; or
//  (2) the usage is crossing above the high threshold and
//      the sensor will be off (i.e. sensor is currently on
//      and has pending clear requests).
//
// Subsequent crossings of the high threshold value do not cause
// any triggers unless the usage becomes less than the low threshold.
//
// The sensor will be cleared if:
//  (1) the usage is crossing below the low threshold and
//      the sensor is currently on and no pending
//      clear requests; or
//  (2) the usage is crossing below the low threshold and
//      the sensor will be on (i.e. sensor is currently off
//      and has pending trigger requests).
//
// Subsequent crossings of the low threshold value do not cause
// any clears unless the usage becomes greater than or equal
// to the high threshold.
//
// If the current level is between high and low threhsold, no change.
//
```

### 名前(function name)
```
void SensorInfo::set_gauge_sensor_level(MemoryUsage usage, ThresholdSupport* high_low_threshold) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(high_low_threshold->is_high_threshold_supported(), "just checking");
	
  {- -------------------------------------------
  (1) 現在の使用量が, high threshold を上回っているかどうか (以下の is_over_high), 
      及び, low threshold を下回っているかどうか (以下の is_below_low) を調べておく
      ---------------------------------------- -}

	  bool is_over_high = high_low_threshold->is_high_threshold_crossed(usage);
	  bool is_below_low = high_low_threshold->is_low_threshold_crossed(usage);
	
	  assert(!(is_over_high && is_below_low), "Can't be both true");
	
  {- -------------------------------------------
  (1) もし, high threshold を上回っていて, かつ
      今回 trigger 通知を生成しても trigger 通知が連続しないのであれば, 
      新しい trigger 通知を生成する (_pending_trigger_count フィールドをインクリメントする).
      (なお, この時点で未通知の clear 通知があった場合, それは消しておく (trigger 通知を出す以上 sensor の状態は on になるので))
      (ついでに, _usage フィールドに現在のメモリ使用量の情報を記録しておく)
      ---------------------------------------- -}

	  if (is_over_high &&
	        ((!_sensor_on && _pending_trigger_count == 0) ||
	         _pending_clear_count > 0)) {
	    // low memory detected and need to increment the trigger pending count
	    // if the sensor is off or will be off due to _pending_clear_ > 0
	    // Request to trigger the sensor
	    _pending_trigger_count++;
	    _usage = usage;
	
	    if (_pending_clear_count > 0) {
	      // non-zero pending clear requests indicates that there are
	      // pending requests to clear this sensor.
	      // This trigger request needs to clear this clear count
	      // since the resulting sensor flag should be on.
	      _pending_clear_count = 0;
	    }

  {- -------------------------------------------
  (1) 逆に, low threshold を下回っていて, かつ
      今回 clear 通知を生成しても clear が連続しないのであれば, 
      新しい clear 通知を生成する (_pending_clear_count フィールドをインクリメントする).
      ---------------------------------------- -}

	  } else if (is_below_low &&
	               ((_sensor_on && _pending_clear_count == 0) ||
	                (_pending_trigger_count > 0 && _pending_clear_count == 0))) {
	    // memory usage returns below the threshold
	    // Request to clear the sensor if the sensor is on or will be on due to
	    // _pending_trigger_count > 0 and also no clear request
	    _pending_clear_count++;
	  }
	}
	
```


