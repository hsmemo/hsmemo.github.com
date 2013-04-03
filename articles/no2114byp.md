---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotRuntimeMBean 
---
[Up](nouYTgvZOF.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotRuntimeMBean 

--- 
## 概要(Summary)
sun.management.HotspotRuntime クラスの各メソッドは,
RuntimeService クラス内の値を取得するか,
あるいは runtime に関連した PerfData の値を取得するだけ.

## 備考(Notes)
(なお, このクラスは JSR-174 には存在しない Sun Microsystems の独自拡張機能)

## 処理の流れ (概要)(Execution Flows : Summary)
### sun.management.HotspotRuntimeMBean.getSafepointCount() の処理
```
sun.management.HotspotRuntime.getSafepointCount()
-> sun.management.VMManagementImpl.getSafepointCount()
   -> Java_sun_management_VMManagementImpl_getSafepointCount()
      -> jmm_GetLongAttribute()  (JMM_SAFEPOINT_COUNT を引数として呼び出される)
         -> get_long_attribute()
            -> RuntimeService::safepoint_count()
```

### sun.management.HotspotRuntimeMBean.getTotalSafepointTime() の処理
```
sun.management.HotspotRuntime.getTotalSafepointTime()
-> sun.management.VMManagementImpl.getTotalSafepointTime()
   -> Java_sun_management_VMManagementImpl_getTotalSafepointTime()
      -> jmm_GetLongAttribute()  (JMM_TOTAL_STOPPED_TIME_MS を引数として呼び出される)
         -> get_long_attribute()
            -> RuntimeService::safepoint_time_ms()
```

### sun.management.HotspotRuntimeMBean.getSafepointSyncTime() の処理
```
sun.management.HotspotRuntime.getSafepointSyncTime()
-> sun.management.VMManagementImpl.getSafepointSyncTime()
   -> Java_sun_management_VMManagementImpl_getSafepointSyncTime()
      -> jmm_GetLongAttribute()  (JMM_TOTAL_SAFEPOINTSYNC_TIME_MS を引数として呼び出される)
         -> get_long_attribute()
            -> RuntimeService::safepoint_sync_time_ms()
```

### sun.management.HotspotRuntimeMBean.getInternalRuntimeCounters() の処理
```
sun.management.HotspotRuntime.getInternalRuntimeCounters()
-> sun.management.VMManagementImpl.getInternalCounters()
   -> (See: [here](norvN2FPOq.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)






