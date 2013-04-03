---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MonitorContendedEnter イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MonitorContendedEnter イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略) (See: [here](no96623Ns.html) and [here](no5248b4E.html) for details)
-> ObjectMonitor::enter()
   -> JvmtiExport::post_monitor_contended_enter()
      -> (登録されているコールバックを呼び出す)
```

## 処理の流れ (詳細)(Execution Flows : Details)






