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
<div class="flow-abst"><pre>
sun.management.HotspotRuntime.getSafepointCount()
-&gt; sun.management.VMManagementImpl.getSafepointCount()
   -&gt; Java_sun_management_VMManagementImpl_getSafepointCount()
      -&gt; jmm_GetLongAttribute()  (JMM_SAFEPOINT_COUNT を引数として呼び出される)
         -&gt; get_long_attribute()
            -&gt; RuntimeService::safepoint_count()
</pre></div>

### sun.management.HotspotRuntimeMBean.getTotalSafepointTime() の処理
<div class="flow-abst"><pre>
sun.management.HotspotRuntime.getTotalSafepointTime()
-&gt; sun.management.VMManagementImpl.getTotalSafepointTime()
   -&gt; Java_sun_management_VMManagementImpl_getTotalSafepointTime()
      -&gt; jmm_GetLongAttribute()  (JMM_TOTAL_STOPPED_TIME_MS を引数として呼び出される)
         -&gt; get_long_attribute()
            -&gt; RuntimeService::safepoint_time_ms()
</pre></div>

### sun.management.HotspotRuntimeMBean.getSafepointSyncTime() の処理
<div class="flow-abst"><pre>
sun.management.HotspotRuntime.getSafepointSyncTime()
-&gt; sun.management.VMManagementImpl.getSafepointSyncTime()
   -&gt; Java_sun_management_VMManagementImpl_getSafepointSyncTime()
      -&gt; jmm_GetLongAttribute()  (JMM_TOTAL_SAFEPOINTSYNC_TIME_MS を引数として呼び出される)
         -&gt; get_long_attribute()
            -&gt; RuntimeService::safepoint_sync_time_ms()
</pre></div>

### sun.management.HotspotRuntimeMBean.getInternalRuntimeCounters() の処理
<div class="flow-abst"><pre>
sun.management.HotspotRuntime.getInternalRuntimeCounters()
-&gt; sun.management.VMManagementImpl.getInternalCounters()
   -&gt; (See: <a href="norvN2FPOq.html">here</a> for details)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)






