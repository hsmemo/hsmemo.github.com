---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： デッドロック検出用の処理 (java.lang.management.ThreadMXBean.findDeadlockedThreads() および java.lang.management.ThreadMXBean.findMonitorDeadlockedThreads() の処理)  
---
[Up](noMz-1isvk.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： デッドロック検出用の処理 (java.lang.management.ThreadMXBean.findDeadlockedThreads() および java.lang.management.ThreadMXBean.findMonitorDeadlockedThreads() の処理)  

--- 
## 概要(Summary)
(See: JSR-174)

## 処理の流れ (概要)(Execution Flows : Summary)
```
sun.management.ThreadImpl.findDeadlockedThreads()
-> sun.management.ThreadImpl.findDeadlockedThreads0()
   -> Java_sun_management_ThreadImpl_findDeadlockedThreads0()
      -> jmm_FindDeadlockedThreads()        (<= なお, 呼び出し側では jmm interface の FindDeadlocks() を呼び出している. 少し名称がずれているので注意.)
         -> find_deadlocks()
            -> VMThread::execute()
               -> (See: [here](no2935qaz.html) for details)
                  -> VM_FindDeadlocks::doit()
                     -> ThreadService::find_deadlocks_at_safepoint()
            -> DeadlockCycle::threads()
```

```
sun.management.ThreadImpl.findMonitorDeadlockedThreads()
-> sun.management.ThreadImpl.findMonitorDeadlockedThreads0()
   -> Java_sun_management_ThreadImpl_findMonitorDeadlockedThreads0()
      -> jmm_FindMonitorDeadlockedThreads() (<= なお, 呼び出し側では jmm interface の FindCircularBlockedThreads() を呼び出している. 少し名称がずれているので注意.)
         -> find_deadlocks()
            -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)







