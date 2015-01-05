---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetOwnedMonitorInfo() 及び GetOwnedMonitorStackDepthInfo() の処理  
---
[Up](no_DXQUxpU.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： GetOwnedMonitorInfo() 及び GetOwnedMonitorStackDepthInfo() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
### GetOwnedMonitorInfo() の処理
<div class="flow-abst"><pre>
JvmtiEnv::GetOwnedMonitorInfo()
-&gt; * 対象のスレッドがサスペンドしている場合:
     -&gt; JvmtiEnvBase::get_owned_monitors()
   * 〃 がサスペンドしていない場合:
     -&gt; VMThread::execute()
        -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
           -&gt; VM_GetOwnedMonitorInfo::doit()
              -&gt; JvmtiEnvBase::get_owned_monitors()
</pre></div>

### GetOwnedMonitorStackDepthInfo() の処理
<div class="flow-abst"><pre>
JvmtiEnv::GetOwnedMonitorStackDepthInfo()
-&gt; * 対象のスレッドがサスペンドしている場合:
     -&gt; JvmtiEnvBase::get_owned_monitors()
   * 〃 がサスペンドしていない場合:
     -&gt; VMThread::execute()
        -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
           -&gt; VM_GetOwnedMonitorInfo::doit()
              -&gt; JvmtiEnvBase::get_owned_monitors()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetOwnedMonitorInfo()
See: [here](no2935ckC.html) for details
### JvmtiEnvBase::get_owned_monitors()
See: [here](no2935DDV.html) for details
### JvmtiEnvBase::get_locked_objects_in_frame()
(#Under Construction)

### VM_GetOwnedMonitorInfo::doit()
See: [here](no293524O.html) for details
### JvmtiEnv::GetOwnedMonitorStackDepthInfo()
See: [here](no2935puI.html) for details






