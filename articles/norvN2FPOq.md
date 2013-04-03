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
```
sun.management.HotspotMemory.getInternalMemoryCounters()
-> sun.management.VMManagementImpl.getInternalCounters()
   -> sun.management.VMManagementImpl.getPerfInstrumentation()
      -> sun.misc.Perf.GetPerfAction()
      -> sun.misc.Perf.attach()
         -> sun.misc.Perf.attachImpl()
            -> sun.misc.Perf.attach()
               -> Perf_Attach()
```

## 処理の流れ (詳細)(Execution Flows : Details)






