---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： オブジェクト (Object) ： GetObjectMonitorUsage() の処理  
---
[Up](noIBBpK5V0.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： オブジェクト (Object) ： GetObjectMonitorUsage() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JvmtiEnv::GetObjectMonitorUsage()
-&gt; JvmtiEnvBase::get_object_monitor_usage()
-&gt; VMThread::execute()
   -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
      -&gt; VM_GetObjectMonitorUsage::doit()
         -&gt; JvmtiEnvBase::get_object_monitor_usage()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetObjectMonitorUsage()
See: [here](no29353rt.html) for details
### JvmtiEnvBase::get_object_monitor_usage()
(#Under Construction)

### VM_GetObjectMonitorUsage::doit()
See: [here](no29352_C.html) for details






