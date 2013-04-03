---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadEnd イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ThreadEnd イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(スレッドの終了処理) (See: [here](no2935w3j.html) for details)
-> JavaThread::exit()
   -> JvmtiExport::post_thread_end()
      -> (登録されているコールバックを呼び出す)

(HotSpot の正常終了処理) (See: [here](no3059oro.html) for details)
-> Threads::destroy_vm()
   -> before_exit()
      -> JvmtiExport::post_thread_end()
         -> (同上)

(略) (?? 使われていない #TODO)
-> JVM_Exit()
   -> before_exit()
      -> (同上)

(java.lang.System.exit() による終了処理) (See: [here](no2935jtd.html) for details)
-> JVM_Halt()
   -> before_exit()
      -> (同上)

(略)
-> CompileBroker::handle_full_code_cache()
   -> before_exit()  (ただし, 呼び出すのは #ifndef PRODUCT 時のみ)
      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)






