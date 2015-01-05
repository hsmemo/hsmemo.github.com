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
<div class="flow-abst"><pre>
sun.management.ThreadImpl.findDeadlockedThreads()
-&gt; sun.management.ThreadImpl.findDeadlockedThreads0()
   -&gt; Java_sun_management_ThreadImpl_findDeadlockedThreads0()
      -&gt; jmm_FindDeadlockedThreads()        (&lt;= なお, 呼び出し側では jmm interface の FindDeadlocks() を呼び出している. 少し名称がずれているので注意.)
         -&gt; find_deadlocks()
            -&gt; VMThread::execute()
               -&gt; (See: <a href="no2935qaz.html">here</a> for details)
                  -&gt; VM_FindDeadlocks::doit_prologue()
                  -&gt; VM_FindDeadlocks::doit()
                     -&gt; ThreadService::find_deadlocks_at_safepoint()
            -&gt; DeadlockCycle::threads()
</pre></div>

<div class="flow-abst"><pre>
sun.management.ThreadImpl.findMonitorDeadlockedThreads()
-&gt; sun.management.ThreadImpl.findMonitorDeadlockedThreads0()
   -&gt; Java_sun_management_ThreadImpl_findMonitorDeadlockedThreads0()
      -&gt; jmm_FindMonitorDeadlockedThreads() (&lt;= なお, 呼び出し側では jmm interface の FindCircularBlockedThreads() を呼び出している. 少し名称がずれているので注意.)
         -&gt; find_deadlocks()
            -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)







