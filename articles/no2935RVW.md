---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： GetFrameLocation() の処理  
---
[Up](noa-wMJL5x.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： GetFrameLocation() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JvmtiEnv::GetFrameLocation()
-&gt; * 対象のスレッドがサスペンドしている場合:
     -&gt; JvmtiEnvBase::get_frame_location()
   * 〃 がサスペンドしていない場合:
     -&gt; VMThread::execute()
        -&gt; (略) (See: <a href="no2935qaz.html">here</a> for details)
           -&gt; VM_GetFrameLocation::doit()
              -&gt; JvmtiEnvBase::get_frame_location()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::GetFrameLocation()
See: [here](no2935efc.html) for details
### JvmtiEnvBase::get_frame_location()
(#Under Construction)

### VM_GetFrameLocation::doit()
See: [here](no2935rpi.html) for details






