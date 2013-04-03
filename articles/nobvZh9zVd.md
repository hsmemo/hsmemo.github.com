---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ResourceExhausted イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ResourceExhausted イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略) (See: [here](no2935KMw.html) for details)
-> JVM_StartThread()
   -> JvmtiExport::post_resource_exhausted()
      -> (登録されているコールバックを呼び出す)

(略) (See: [here](no28916Q0G.html) for details)
-> CollectedHeap::common_mem_allocate_noinit()
   -> JvmtiExport::post_resource_exhausted()
      -> (登録されているコールバックを呼び出す)

(略) (See: [here](no28916-pc.html) for details)
-> CollectedHeap::common_permanent_mem_allocate_noinit()
   -> JvmtiExport::post_resource_exhausted()
      -> (登録されているコールバックを呼び出す)
```

## 処理の流れ (詳細)(Execution Flows : Details)






