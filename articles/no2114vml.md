---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotThreadMBean  
---
[Up](nouYTgvZOF.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.HotspotThreadMBean  

--- 
## 概要(Summary)
(#Under Construction)

## 備考(Notes)
(なお, このクラスは JSR-174 には存在しない Sun Microsystems の独自拡張機能)

## 処理の流れ (概要)(Execution Flows : Summary)
### sun.management.HotspotThreadMBean.getInternalThreadCount() の処理
```
sun.management.HotspotThread.getInternalThreadCount()
-> Java_sun_management_HotspotThread_getInternalThreadCount()
   -> jmm_GetLongAttribute()  (JMM_VM_THREAD_COUNT を引数として呼び出される)
      -> get_long_attribute()
         -> get_vm_thread_count()
            -> Threads::threads_do()
               -> (See: )
                  -> VmThreadCountClosure::do_thread()
```

### sun.management.HotspotThreadMBean.getInternalThreadCpuTimes() の処理
```
sun.management.HotspotThread.getInternalThreadCpuTimes()
-> sun.management.HotspotThread.getInternalThreadTimes0()
   -> Java_sun_management_HotspotThread_getInternalThreadTimes0()
      -> jmm_GetInternalThreadTimes()
         -> Threads::threads_do()
            -> (See: )
               -> ThreadTimesClosure::do_thread()
```

### sun.management.HotspotThreadMBean.getInternalThreadingCounters() の処理
```
sun.management.HotspotThread.getInternalThreadingCounters()
-> sun.management.VMManagementImpl.getInternalCounters()
   -> (See: [here](norvN2FPOq.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### Java_sun_management_HotspotThread_getInternalThreadCount()
See: [here](no2114I6q.html) for details
### get_vm_thread_count()
See: [here](no2114VEx.html) for details
### VmThreadCountClosure::do_thread()
See: [here](no2114HOA.html) for details

### sun.management.HotspotThread.getInternalThreadCpuTimes()
(#Under Construction)
See: [here](no2114hbY.html) for details
### Java_sun_management_HotspotThread_getInternalThreadTimes0()
See: [here](no2114UYG.html) for details
### jmm_GetInternalThreadTimes()
See: [here](no2114hiM.html) for details
### ThreadTimesClosure::do_thread()
See: [here](no211472Y.html) for details

### sun.management.HotspotThread.getInternalThreadingCounters()
See: [here](no2114ule.html) for details
### sun.management.VMManagementImpl.getInternalCounters()
(#Under Construction)
See: [here](no21147vk.html) for details







