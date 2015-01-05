---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： StopThread() の処理
---
[Up](no_DXQUxpU.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スレッド (Thread) ： StopThread() の処理

--- 
## 概要(Summary)
処理は VM_ThreadStop を用いて実装されている.
最終的には JavaThread::send_thread_stop() で対象のスレッド内に pending exception を埋め込むことで実現される.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JvmtiEnv::StopThread()
-&gt; Thread::send_async_exception()
   -&gt; VMThread::execute()
      -&gt; (See: <a href="no2935qaz.html">here</a> for details)
         -&gt; VM_ThreadStop::doit()
            -&gt; JavaThread::send_thread_stop()
               -&gt; JavaThread::set_pending_async_exception()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::StopThread()
See: [here](no28916Ijl.html) for details
### Thread::send_async_exception()
See: [here](no2114LNS.html) for details
### VM_ThreadStop::doit()
See: [here](no2114YXY.html) for details
### JavaThread::send_thread_stop()
See: [here](no2114lhe.html) for details
### JavaThread::set_pending_async_exception()
See: [here](no2114yrk.html) for details







