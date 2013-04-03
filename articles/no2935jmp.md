---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 監視されるフィールド (Watched Field) ： SetFieldAccessWatch(), ClearFieldAccessWatch(), SetFieldModificationWatch(), ClearFieldModificationWatch() の処理  
---
[Up](no-MaF_wEn.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 監視されるフィールド (Watched Field) ： SetFieldAccessWatch(), ClearFieldAccessWatch(), SetFieldModificationWatch(), ClearFieldModificationWatch() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)

## 備考(Notes)
* フィールドの watch point 機能を利用するためには,
  これらの関数を呼び出すことに加えて,
  通常のイベント通知設定処理(callback の設定,通知の有効化)も行っておく必要がある. (See: [here](no2935C7Z.html) for details)

* watch point 機能を有効にした場合 interp_only_mode になる. (See: [here](no3059eFS.html) for details)

* watch point capability を取得した場合に true になる JvmtiExport::can_* フィールドは以下の通り. (#TODO)


## 処理の流れ (概要)(Execution Flows : Summary)
### watch point 関係の capability の有効化処理 (AddCapabilities() の処理)
(See: [here](no2935trw.html) for details)

なお watch point 用の capability を取得すると,
(JvmtiExport::can_*() が変更されることに加えて)
interp_only_mode 用に様々な最適化オプションが無効になる (See: [here](no3059eFS.html) for details).

### watch point 関係の callback の設定処理 (SetEventCallbacks() の処理)
(See: [here](no2935C7Z.html) for details)

### watch point event の有効化処理 (SetEventNotificationMode() の処理)
(See: [here](no2935C7Z.html) for details)

### 監視対象の指定処理 (SetFieldAccessWatch(), ClearFieldAccessWatch(), SetFieldModificationWatch(), ClearFieldModificationWatch() の処理)
(処理の流れは4つともほぼ同じなので, 以下では JvmtiEnv::SetFieldAccessWatch() のみ記述)

```
JvmtiEnv::SetFieldAccessWatch()
-> JvmtiEventController::change_field_watch()
   -> JvmtiEventControllerPrivate::change_field_watch()
      -> JvmtiEventControllerPrivate::recompute_enabled()
         -> (略) (See: [here](no2935C7Z.html) for details)
```

### watch point のフック処理
```
* Template Interpreter の場合:

  TemplateTable::getfield_or_static() もしくは TemplateTable::fast_accessfield() が生成するコード
  -> TemplateTable::jvmti_post_field_access() が生成するコード
     -> JvmtiExport::can_post_field_access()
     -> InterpreterRuntime::post_field_access()
        -> JvmtiExport::post_field_access()

  TemplateTable::putfield_or_static()
  -> TemplateTable::jvmti_post_field_mod() が生成するコード
     -> JvmtiExport::can_post_field_modification()
     -> InterpreterRuntime::post_field_modification()
        -> JvmtiExport::post_raw_field_modification()

  TemplateTable::fast_storefield()
  -> TemplateTable::jvmti_post_fast_field_mod() が生成するコード
     -> JvmtiExport::can_post_field_modification()
     -> InterpreterRuntime::post_field_modification()
        -> (同上)

* C++ Interpreter の場合:

  BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
  -> InterpreterRuntime::post_field_access()
     -> (同上)

  BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
  -> InterpreterRuntime::post_field_modification()
     -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::SetFieldAccessWatch()
See: [here](no2935bJV.html) for details
### fieldDescriptor::set_is_field_access_watched()
See: [here](no2935Pyt.html) for details
### AccessFlags::set_is_field_access_watched()
See: [here](no2935c8z.html) for details
### JvmtiEnvBase::update_klass_field_access_flag()
See: [here](no2935oaP.html) for details
### JvmtiEventController::change_field_watch()
See: [here](no2935OGD.html) for details
### JvmtiEventControllerPrivate::change_field_watch()
See: [here](no2935bQJ.html) for details

### JvmtiEnv::ClearFieldAccessWatch()
See: [here](no2935oTb.html) for details
### JvmtiEnv::SetFieldModificationWatch()
See: [here](no29351dh.html) for details
### JvmtiEnv::ClearFieldModificationWatch()
See: [here](no2935Con.html) for details

### TemplateTable::jvmti_post_field_access() (sparc の場合)
(#Under Construction)
### TemplateTable::jvmti_post_field_access() (x86_64 の場合)
(#Under Construction)
### JvmtiExport::can_post_field_access()
(#Under Construction)
### InterpreterRuntime::post_field_access()
(#Under Construction)
### JvmtiExport::post_field_access()
(#Under Construction)

### TemplateTable::jvmti_post_field_mod() (sparc の場合)
(#Under Construction)
### TemplateTable::jvmti_post_field_mod() (x86_64 の場合)
(#Under Construction)
### JvmtiExport::can_post_field_modification()
(#Under Construction)
### InterpreterRuntime::post_field_modification()
(#Under Construction)
### JvmtiExport::post_raw_field_modification()
(#Under Construction)

### TemplateTable::jvmti_post_fast_field_mod() (sparc の場合)
(#Under Construction)
### TemplateTable::jvmti_post_fast_field_mod() (x86_64 の場合)
(#Under Construction)







