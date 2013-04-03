---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： SingleStep イベントの処理  
---
[Up](no29359PS.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： イベント管理 (Event Management) ： 各イベントの通知処理 ： SingleStep イベントの処理  

--- 
## 概要(Summary)
SetEventCallbacks() と SetEventNotificationMode() によって SingleStep event が有効化されると,
VM_ChangeSingleStep::doit() により SingleStep モードが始まる.

その後, SetEventCallbacks() か SetEventNotificationMode() によって SingleStep event が disabled にされると,
VM_ChangeSingleStep::doit() により SingleStep モードが終了する.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.cpp))
        // If running in fullspeed mode, single stepping is implemented
        // as follows: first, the interpreter does not dispatch to
        // compiled code for threads that have single stepping enabled;
        // second, we deoptimize all methods on the thread's stack when
        // interpreted-only mode is enabled the first time for a given
        // thread (nothing to do if no Java frames yet).
```

## 備考(Notes)
* Template Interpreter の場合,
  SingleStep モードは常に safepoint 停止用の dispatch table で実行される.
  
  (通常は InterpreterRuntime::at_safepoint() が終了する時に safepoint チェックがあり,
  その中で TemplateInterpreter::ignore_safepoints() が呼ばれて dispatch table を元に戻してしまう.
  しかし JvmtiExport::should_post_single_step() が true の時だけは戻さないので,
  SingleStep モードではずっと safepoint 用のテーブルのままになる).

* この機能を有効にした場合 interp_only_mode になる (See: [here](no3059eFS.html) for details).

* singlestep capability を取得した場合に true になる JvmtiExport::can_* フィールドは以下の通り. (#TODO)


## 処理の流れ (概要)(Execution Flows : Summary)
### singlestep 関係の capability の有効化処理 (AddCapabilities() の処理)
(See: [here](no2935trw.html) for details)

なお singlestep 用の capability を取得すると,
(JvmtiExport::can_*() が変更されることに加えて)
interp_only_mode 用に様々な最適化オプションが無効になる (See: [here](no3059eFS.html) for details).

### singlestep 関係のコールバックの設定処理 (SetEventCallbacks() の処理)
(なお breakpoint の場合には,
JvmtiEventControllerPrivate::recompute_enabled() の中で
JvmtiEnvThreadState::reset_current_location() が行われる. #TODO)

```
JvmtiEnv::SetEventCallbacks()
-> (略) (See: [here](no2935C7Z.html) for details)
   -> JvmtiEventControllerPrivate::recompute_enabled()
      -> VMThread::execute()
         -> (略) (See: [here](no2935qaz.html) for details)
            -> VM_ChangeSingleStep::doit()
               -> JvmtiEventControllerPrivate::set_should_post_single_step()
                  -> JvmtiExport::set_should_post_single_step()
                     -> Interpreter::notice_safepoints() (をサブクラスがオーバーライドしたもの)
```

### singlestep 関係のイベント通知の有効化処理 (NotificationMode() の処理)
(なお breakpoint の場合には,
JvmtiEventControllerPrivate::recompute_enabled() の中で
JvmtiEnvThreadState::reset_current_location() が行われる. #TODO)

```
JvmtiEnv::SetEventNotificationMode()
-> (略) (See: [here](no2935C7Z.html) for details)
   -> JvmtiEventControllerPrivate::recompute_enabled()
      -> VMThread::execute()
         -> (略) (See: [here](no2935qaz.html) for details)
            -> VM_ChangeSingleStep::doit()
               -> (同上)
```

### singlestep モードでの実行処理
```
* Template Interpreter の場合:

  _safept_table の各エントリ
  -> (略) (See: [here](no7882rhh.html) for details)
      -> InterpreterRuntime::at_safepoint()
         -> JvmtiExport::at_single_stepping_point()
            -> JvmtiExport::post_single_step()

* C++ Interpreter の場合:

  DEBUGGER_SINGLE_STEP_NOTIFY() マクロ
  -> InterpreterRuntime::at_safepoint()
     -> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### VM_ChangeSingleStep::doit()
See: [here](no2935azx.html) for details
### JvmtiEventControllerPrivate::set_should_post_single_step()
See: [here](no2935M9A.html) for details
### JvmtiExport::set_should_post_single_step()
(#Under Construction)

### TemplateInterpreter::notice_safepoints()
See: [here](no7882qZm.html) for details
### CppInterpreter::notice_safepoints()
See: [here](no7882dPg.html) for details

### InterpreterRuntime::at_safepoint()
See: [here](no7882eXb.html) for details
### JvmtiExport::at_single_stepping_point()
See: [here](no2935b7s.html) for details
### JvmtiExport::post_single_step()
See: [here](no2935oFz.html) for details





