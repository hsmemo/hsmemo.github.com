---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： Raw モニター (Raw Monitor) ： CreateRawMonitor(), DestroyRawMonitor(), RawMonitorEnter(), RawMonitorExit(), RawMonitorWait(), RawMonitorNotify(), RawMonitorNotifyAll() の処理  
---
[Up](noCTv9Ipvj.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： Raw モニター (Raw Monitor) ： CreateRawMonitor(), DestroyRawMonitor(), RawMonitorEnter(), RawMonitorExit(), RawMonitorWait(), RawMonitorNotify(), RawMonitorNotifyAll() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 備考(Notes)
まだ JavaThread ができていない状態 (HotSpot が起動中の段階) で RawMonitor が使用された場合のために, 
JvmtiPendingMonitors というクラスが用意されている.

起動中にロックされた RawMonitor は JvmtiPendingMonitors クラス内で管理され,
起動処理の終了後にメインスレッドを表す JavaThread へと引き継がれる.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
* JvmtiEnv::CreateRawMonitor() の処理

  JvmtiEnv::CreateRawMonitor()
  -&gt; JvmtiRawMonitor::JvmtiRawMonitor()

* JvmtiEnv::DestroyRawMonitor() の処理

  JvmtiEnv::DestroyRawMonitor()
  -&gt; JvmtiPendingMonitors::destroy()     (&lt;= HotSpot の起動中であれば呼び出す)
  -&gt; JvmtiRawMonitor::~JvmtiRawMonitor()

* JvmtiEnv::RawMonitorEnter() の処理

  JvmtiEnv::RawMonitorEnter()
  -&gt; * HotSpot の起動中の場合:
       -&gt; JvmtiPendingMonitors::enter()
     * 起動中ではない場合:
       -&gt; JvmtiRawMonitor::raw_enter()
          -&gt; JvmtiRawMonitor::SimpleEnter()

* JvmtiEnv::RawMonitorExit() の処理

  JvmtiEnv::RawMonitorExit()
  -&gt; * HotSpot の起動中の場合:
       -&gt; JvmtiPendingMonitors::exit()
     * 起動中ではない場合:
       -&gt; JvmtiRawMonitor::raw_exit()
          -&gt; JvmtiRawMonitor::SimpleExit()

* JvmtiEnv::RawMonitorWait() の処理

  JvmtiEnv::RawMonitorWait()
  -&gt; JvmtiRawMonitor::raw_wait()
     -&gt; JvmtiRawMonitor::SimpleWait()

* JvmtiEnv::RawMonitorNotify() の処理

  JvmtiEnv::RawMonitorNotify()
  -&gt; JvmtiRawMonitor::raw_notify()
     -&gt; JvmtiRawMonitor::SimpleNotify()

* JvmtiEnv::RawMonitorNotifyAll() の処理

  JvmtiEnv::RawMonitorNotifyAll()
  -&gt; JvmtiRawMonitor::raw_notifyAll()
     -&gt; JvmtiRawMonitor::SimpleNotify()

* 起動処理後の引き継ぎ処理

  Threads::create_vm()
  -&gt; JvmtiExport::transition_pending_onload_raw_monitors()
     -&gt; JvmtiPendingMonitors::transition_raw_monitors()

</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::CreateRawMonitor()
See: [here](no2935tHB.html) for details
### JvmtiRawMonitor::JvmtiRawMonitor()
See: [here](no29356RH.html) for details

### JvmtiEnv::DestroyRawMonitor()
See: [here](no2935HcN.html) for details
### JvmtiPendingMonitors::destroy()
See: [here](no2935u6f.html) for details
### JvmtiRawMonitor::~JvmtiRawMonitor()
See: [here](no2935UmT.html) for details

### JvmtiEnv::RawMonitorEnter()
See: [here](no2935hwZ.html) for details
### JvmtiPendingMonitors::enter()
See: [here](no29357Em.html) for details
### JvmtiRawMonitor::raw_enter()
(#Under Construction)
See: [here](no2935IPs.html) for details
### JvmtiRawMonitor::SimpleEnter()
(#Under Construction)


### JvmtiEnv::RawMonitorExit()
See: [here](no2935VZy.html) for details
### JvmtiPendingMonitors::exit()
See: [here](no2935HjB.html) for details
### JvmtiRawMonitor::raw_exit()
See: [here](no2935UtH.html) for details
### JvmtiRawMonitor::SimpleExit()
See: [here](no2935h3N.html) for details

### JvmtiEnv::RawMonitorWait()
See: [here](no2935uBU.html) for details
### JvmtiRawMonitor::raw_wait()
(#Under Construction)
See: [here](no29357La.html) for details
### JvmtiRawMonitor::SimpleWait()
(#Under Construction)


### JvmtiEnv::RawMonitorNotify()
See: [here](no2935IWg.html) for details
### JvmtiRawMonitor::raw_notify()
See: [here](no2935iqs.html) for details
### JvmtiRawMonitor::SimpleNotify()
See: [here](no2935h-B.html) for details

### JvmtiEnv::RawMonitorNotifyAll()
See: [here](no2935Vgm.html) for details
### JvmtiRawMonitor::raw_notifyAll()
See: [here](no2935v0y.html) for details
### JvmtiExport::transition_pending_onload_raw_monitors()
See: [here](no17119Nfm.html) for details
### JvmtiPendingMonitors::transition_raw_monitors()
See: [here](no17119aps.html) for details






