---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetCurrentContendedMonitor() の処理  
---
[Up](no_DXQUxpU.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetCurrentContendedMonitor() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JvmtiEnv::GetCurrentContendedMonitor()
-&gt; * 対象のスレッドがサスペンドしている場合:
     -&gt; JvmtiEnvBase::get_current_contended_monitor()
   * 〃 がサスペンドしていない場合:
     -&gt; VMThread::execute()
        -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
           -&gt; VM_GetCurrentContendedMonitor::doit()
              -&gt; JvmtiEnvBase::get_current_contended_monitor()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetCurrentContendedMonitor()
See: [here](no2935deV.html) for details
### JvmtiEnvBase::get_current_contended_monitor()
(#Under Construction)

### VM_GetCurrentContendedMonitor::doit()
See: [here](no2935qob.html) for details






