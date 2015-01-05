---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotMemoryMBean
---
[Up](nouYTgvZOF.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotMemoryMBean

--- 
## 概要(Summary)
sun.management.HotspotMemory クラスのメソッドは,
メモリに関連した PerfData の値を取得するだけ.

## 備考(Notes)
(なお, このクラスは JSR-174 には存在しない Sun Microsystems の独自拡張機能)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
sun.management.HotspotMemory.getInternalMemoryCounters()
-&gt; sun.management.VMManagementImpl.getInternalCounters()
   -&gt; sun.management.VMManagementImpl.getPerfInstrumentation()
      -&gt; sun.misc.Perf.GetPerfAction()
      -&gt; sun.misc.Perf.attach()
         -&gt; sun.misc.Perf.attachImpl()
            -&gt; sun.misc.Perf.attach()
               -&gt; Perf_Attach()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






