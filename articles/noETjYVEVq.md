---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadStart イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadStart イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> JNI_CreateJavaVM()
   -> JvmtiExport::post_thread_start()
      -> (登録されているコールバックを呼び出す)

(略) (See: [here](no7882jgS.html) for details)
-> JavaThread::run()
   -> JvmtiExport::post_thread_start()
      -> (同上)

(略) (See: [here](noxegGjntv.html) for details)
-> jni_AttachCurrentThread()
   -> attach_current_thread()
      -> JvmtiExport::post_thread_start()
         -> (同上)

(略) (See: [here](noxegGjntv.html) for details)
-> jni_AttachCurrentThreadAsDaemon()
   -> attach_current_thread()
      -> JvmtiExport::post_thread_start()
         -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






