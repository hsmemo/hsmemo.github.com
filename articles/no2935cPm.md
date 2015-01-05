---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： SetThreadLocalStorage() 及び GetThreadLocalStorage() の処理  
---
[Up](no_DXQUxpU.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： SetThreadLocalStorage() 及び GetThreadLocalStorage() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

値は処理対象のスレッドに対応する JvmtiEnvThreadState オブジェクト内に格納される
(See: JvmtiEnvThreadState).

## 処理の流れ (概要)(Execution Flows : Summary)
### SetThreadLocalStorage() の処理
<div class="flow-abst"><pre>
JvmtiEnv::SetThreadLocalStorage()
-&gt; JvmtiEnvThreadState::set_agent_thread_local_storage_data()
</pre></div>

### GetThreadLocalStorage() の処理
<div class="flow-abst"><pre>
JvmtiEnv::GetThreadLocalStorage()
-&gt; JvmtiEnvThreadState::get_agent_thread_local_storage_data()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::SetThreadLocalStorage()
See: [here](no17119oJW.html) for details
### JvmtiEnvThreadState::set_agent_thread_local_storage_data()
See: [here](no171191Tc.html) for details
### JvmtiEnv::GetThreadLocalStorage()
See: [here](no1776676a.html) for details
### JvmtiEnvThreadState::get_agent_thread_local_storage_data()
See: [here](no17766WQu.html) for details





