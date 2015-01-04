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
(See: [here](noIvSV0NZj.html) for details)
-> SystemDictionary::parse_stream()
   -> JvmtiExport::post_class_load()
      -> (登録されているコールバックを呼び出す)

(See: [here](noIvSV0NZj.html) for details)
-> SystemDictionary::resolve_instance_class_or_null()
   -> JvmtiExport::post_class_load()
      -> (同上)

(See: [here](noIvSV0NZj.html) for details)
-> SystemDictionary::define_instance_class()
   -> JvmtiExport::post_class_load()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






