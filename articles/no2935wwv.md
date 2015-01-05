---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： VMObjectAlloc イベントの処理  
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： VMObjectAlloc イベントの処理  

--- 
## 概要(Summary)
VMObjectAlloc の通知処理は, 直接 JvmtiExport::post_vm_object_alloc() を呼び出すパスの他に,
JvmtiVMObjectAllocEventCollector オブジェクトを用いたパスが存在する.

JvmtiVMObjectAllocEventCollector を使用する場合,
まず通知するイベントの情報を JvmtiVMObjectAllocEventCollector オブジェクト内に溜めた後,
JvmtiVMObjectAllocEventCollector のデストラクタ内で通知処理が行われる
(See: JvmtiVMObjectAllocEventCollector).


## 処理の流れ (概要)(Execution Flows : Summary)
### JvmtiVMObjectAllocEventCollector オブジェクト用の記録処理
<div class="flow-abst"><pre>
(略) (no28916Q0G)
-&gt; CollectedHeap::post_allocation_setup_obj()
   -&gt; post_allocation_notify()
      -&gt; JvmtiExport::vm_object_alloc_event_collector()
         -&gt; JvmtiExport::record_vm_internal_object_allocation()
            -&gt; JvmtiVMObjectAllocEventCollector::record_allocation()

(略) (no28916Q0G)
-&gt; CollectedHeap::post_allocation_setup_array()
   -&gt; post_allocation_notify()
      -&gt; (同上)
</pre></div>

### イベントの通知処理
<div class="flow-abst"><pre>
JVM_NewInstance()
-&gt; JvmtiExport::post_vm_object_alloc()
   -&gt; (登録されているコールバックを呼び出す)

JVM_InvokeMethod()
-&gt; JvmtiExport::post_vm_object_alloc()
   -&gt; (同上)

JVM_NewInstanceFromConstructor()
-&gt; JvmtiExport::post_vm_object_alloc()
   -&gt; (同上)

JvmtiVMObjectAllocEventCollector::~JvmtiVMObjectAllocEventCollector()
-&gt; JvmtiExport::post_vm_object_alloc()
   -&gt; (同上)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiVMObjectAllocEventCollector::JvmtiVMObjectAllocEventCollector()
(#Under Construction)
See: [here](no2935961.html) for details
### JvmtiEventCollector::setup_jvmti_thread_state()
See: [here](no2935isW.html) for details
### JvmtiThreadState::set_vm_object_alloc_event_collector()
See: [here](no2935JLp.html) for details

### post_allocation_notify()
See: [here](no344Pdy.html) for details
### JvmtiExport::vm_object_alloc_event_collector()
See: [here](no2935Wcj.html) for details
### JvmtiExport::record_vm_internal_object_allocation()
(#Under Construction)
See: [here](no2935JSd.html) for details
### JvmtiVMObjectAllocEventCollector::record_allocation()
See: [here](no29358HX.html) for details
### JvmtiThreadState::get_vm_object_alloc_event_collector()
See: [here](no2935jf1.html) for details

### JvmtiVMObjectAllocEventCollector::~JvmtiVMObjectAllocEventCollector()
(#Under Construction)
See: [here](no2935vEF.html) for details
### JvmtiEventCollector::unset_jvmti_thread_state()
See: [here](no2935v2c.html) for details






