---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： DataDumpRequest イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： DataDumpRequest イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略) (?? 使われていない #TODO)
-> JVM_DumpAllStacks()
   -> JvmtiExport::post_data_dump()
      -> (登録されているコールバックを呼び出す)

(略) (See: [here](no28916GhX.html) for details)
-> signal_thread_entry()
   -> JvmtiExport::post_data_dump()
      -> (同上)

(略) (See: [here](no3026gMG.html) for details)
-> attach_listener_thread_entry()
   -> data_dump()
      -> JvmtiExport::post_data_dump()
         -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






