---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： Exception イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： Exception イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略) (See: [here](no30593YX.html) for details)
-> InterpreterRuntime::exception_handler_for_exception()
   -> JvmtiExport::post_exception_throw()
      -> (登録されているコールバックを呼び出す)

(略)
-> Runtime1::post_jvmti_exception_throw()
   -> JvmtiExport::post_exception_throw()
      -> (同上)

(略)
-> SharedRuntime::throw_and_post_jvmti_exception()
   -> JvmtiExport::post_exception_throw()
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






