---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassFileLoadHook イベントの処理  
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： ClassFileLoadHook イベントの処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(略)
-> ClassFileParser::parseClassFile()
   -> JvmtiExport::post_class_file_load_hook()
      -> JvmtiClassFileLoadHookPoster::post()
         -> JvmtiClassFileLoadHookPoster::post_all_envs()
            -> JvmtiClassFileLoadHookPoster::post_to_env()
               -> (登録されているコールバックを呼び出す)
```

## 処理の流れ (詳細)(Execution Flows : Details)







