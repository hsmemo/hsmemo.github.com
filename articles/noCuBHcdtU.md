---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MonitorContendedEntered イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MonitorContendedEntered イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(略) (See: <a href="no96623Ns.html">here</a> and <a href="no5248b4E.html">here</a> for details)
-&gt; ObjectMonitor::enter()
   -&gt; JvmtiExport::post_monitor_contended_entered()
      -&gt; (登録されているコールバックを呼び出す)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)






