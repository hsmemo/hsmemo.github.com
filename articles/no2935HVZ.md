---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 全般 (General) ： DisposeEnvironment() の処理  
---
[Up](noh4kNMurg.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 全般 (General) ： DisposeEnvironment() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JvmtiEnv::DisposeEnvironment()
-&gt; JvmtiEnvBase::dispose()
   -&gt; JvmtiEventController::env_dispose()
      -&gt; JvmtiEventControllerPrivate::env_dispose()
         -&gt; JvmtiEnvBase::env_dispose()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::DisposeEnvironment()
See: [here](no2935s_F.html) for details
### JvmtiEnvBase::dispose()
See: [here](no29355JM.html) for details
### JvmtiEventController::env_dispose()
See: [here](no2935GUS.html) for details
### JvmtiEventControllerPrivate::env_dispose()
See: [here](no2935TeY.html) for details
### JvmtiEnvBase::env_dispose()
See: [here](no2935goe.html) for details






