---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassLoad イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassLoad イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略)
-> SystemDictionary::parse_stream()
   -> JvmtiExport::post_class_load()
      -> (登録されているコールバックを呼び出す)

(略)
-> SystemDictionary::resolve_instance_class_or_null()
   -> JvmtiExport::post_class_load()
      -> (同上)

(略)
-> SystemDictionary::define_instance_class()
   -> JvmtiExport::post_class_load()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






