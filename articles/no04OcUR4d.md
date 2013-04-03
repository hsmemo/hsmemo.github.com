---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MethodEntry イベント, 及び MethodExit イベントの処理
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： MethodEntry イベント, 及び MethodExit イベントの処理

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 備考(Notes)
この機能を有効にした場合 interp_only_mode になる. (See: [here](no3059eFS.html) for details)

## 処理の流れ (概要)(Execution Flows : Summary)
### イベントの通知処理
```
InterpreterMacroAssembler::notify_method_entry() が生成するコード
-> JvmtiExport::can_post_interpreter_events()
-> InterpreterRuntime::post_method_entry()
   -> JvmtiExport::post_method_entry()
      -> (登録されているコールバックを呼び出す)

InterpreterMacroAssembler::notify_method_exit() が生成するコード
-> JvmtiExport::can_post_interpreter_events()
-> InterpreterRuntime::post_method_exit()
   -> JvmtiExport::post_method_exit()
      -> (登録されているコールバックを呼び出す)
```

## 処理の流れ (詳細)(Execution Flows : Details)






