---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： イベントの遅延通知 [CompiledMethodLoad, CompiledMethodUnload, DynamicCodeGenerated] 
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： イベントの遅延通知 [CompiledMethodLoad, CompiledMethodUnload, DynamicCodeGenerated] 

--- 
## 概要(Summary)
JVMTI のイベントの中には, 実際にそのイベントが起こった瞬間ではなく, 少し遅れてから通知されるものがある.

(現状の HotSpot の実装では, CompiledMethodLoad, CompiledMethodUnload, DynamicCodeGenerated が遅延配送される模様)


これらのイベントを遅延させているのは, 
イベント通知は JavaThread から行わないといけないが, 
これらのイベントは CompilerThread や VMThread の中で発生するものであるため.

そのため, CompilerThread や VMThread 内ではイベントが発生したという情報だけを記録しておき, 
"ServiceThread" という JavaThread が実際の通知処理を行うことにしている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvmtiImpl.hpp))
    /**
     * When a thread (such as the compiler thread or VM thread) cannot post a
     * JVMTI event itself because the event needs to be posted from a Java
     * thread, then it can defer the event to the Service thread for posting.
     * The information needed to post the event is encapsulated into this class
     * and then enqueued onto the JvmtiDeferredEventQueue, where the Service
     * thread will pick it up and post it.
     *
     * This is currently only used for posting compiled-method-load and unload
     * events, which we don't want posted from the compiler thread.
     */
```

## 備考(Notes)
以下のクラス群によって遅延通知が実現されている.

  * JvmtiDeferredEvent
  * JvmtiDeferredEventQueue
  * JvmtiDeferredEventQueue::QueueNode
  * ServiceThread

すぐに post できないイベントが発生すると,
そのイベントの内容を格納した JvmtiDeferredEvent オブジェクトが作られ,
JvmtiDeferredEventQueue に格納される.

この JvmtiDeferredEventQueue は ServiceThread によって監視されており,
JvmtiDeferredEvent オブジェクトが追加されると ServiceThread がそれを取り出して
JvmtiDeferredEvent::post() を呼び出すことでイベント通知処理を実行する.

## 処理の流れ (概要)(Execution Flows : Summary)
### (1) イベントの生成と ServiceThread の起床処理 (CompiledMethodLoad の場合)
<div class="flow-abst"><pre>
(略) (See: <a href="no3718SNC.html">here</a> for details)
-&gt; ciEnv::register_method()
   -&gt; nmethod::post_compiled_method_load_event()
      -&gt; nmethod::get_and_cache_jmethod_id()
      -&gt; JvmtiDeferredEvent::compiled_method_load_event()
      -&gt; JvmtiDeferredEventQueue::enqueue()

(略) (See: <a href="no293548G.html">here</a> for details)
-&gt; AdapterHandlerLibrary::create_native_wrapper()
   -&gt; nmethod::post_compiled_method_load_event()
      -&gt; (同上)
</pre></div>

### (1) イベントの生成と ServiceThread の起床処理 (CompiledMethodUnload の場合)
<div class="flow-abst"><pre>
(略) (See: ...)
-&gt; nmethod::make_unloaded()
   -&gt; nmethod::post_compiled_method_unload()
      -&gt; JvmtiDeferredEvent::compiled_method_unload_event()
      -&gt; * safepoint 中の場合
           -&gt; JvmtiDeferredEventQueue::add_pending_event()
         * そうではない場合
           -&gt; JvmtiDeferredEventQueue::enqueue()

(略) (See: ...)
-&gt; nmethod::make_not_entrant_or_zombie()
   -&gt; nmethod::post_compiled_method_unload()
      -&gt; (同上)
</pre></div>

### (1) イベントの生成と ServiceThread の起床処理 (DynamicCodeGenerated の場合)
<div class="flow-abst"><pre>
JvmtiExport::post_dynamic_code_generated(const char *name, const void *code_begin, const void *code_end)
-&gt; * HotSpot の起動中の場合:
     -&gt; JvmtiExport::post_dynamic_code_generated_internal()
   * それ以外の場合:
     -&gt; JvmtiDeferredEvent::dynamic_code_generated_event()
     -&gt; JvmtiDeferredEventQueue::enqueue()
</pre></div>

### (2) ServiceThread による通知機能の呼び出し
<div class="flow-abst"><pre>
ServiceThread::service_thread_entry()
-&gt; JvmtiDeferredEventQueue::has_events()
-&gt; JvmtiDeferredEventQueue::dequeue()
-&gt; JvmtiDeferredEvent::post()
   -&gt; * CompiledMethodLoad の場合:
        -&gt; JvmtiExport::post_compiled_method_load()
      * CompiledMethodUnload の場合:
        -&gt; JvmtiExport::post_compiled_method_unload()
      * DynamicCodeGenerated の場合:
        -&gt; JvmtiExport::post_dynamic_code_generated_internal()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### nmethod::post_compiled_method_load_event()
See: [here](no293574n.html) for details
### nmethod::get_and_cache_jmethod_id()
See: [here](no2935IDu.html) for details
### JvmtiDeferredEvent::compiled_method_load_event()
See: [here](no2935VN0.html) for details
### JvmtiDeferredEventQueue::enqueue()
(#Under Construction)
See: [here](no2935hrP.html) for details
### JvmtiDeferredEventQueue::process_pending_events()
(#Under Construction)
See: [here](no29357_b.html) for details

### nmethod::post_compiled_method_unload()
(#Under Construction)
See: [here](no2935IKi.html) for details
### JvmtiDeferredEvent::compiled_method_unload_event()
See: [here](no29357GQ.html) for details

### JvmtiExport::post_dynamic_code_generated(const char *name, const void *code_begin, const void *code_end)
(#Under Construction)
See: [here](no2935Vbc.html) for details
### JvmtiExport::post_dynamic_code_generated_internal()
(#Under Construction)

### JvmtiDeferredEvent::dynamic_code_generated_event()
See: [here](no2935vvo.html) for details

### ServiceThread::service_thread_entry()
See: [here](no2935HXD.html) for details
### JvmtiDeferredEventQueue::has_events()
See: [here](no2935UhJ.html) for details
### JvmtiDeferredEventQueue::dequeue()
(#Under Construction)
See: [here](no2935u1V.html) for details
### JvmtiDeferredEvent::post()
See: [here](no2935u8J.html) for details






