---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 早期復帰の強制 (Force Early Return) ： ForceEarlyReturn*() の処理  
---
[Up](nouYp9PhYI.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： 早期復帰の強制 (Force Early Return) ： ForceEarlyReturn*() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)
(なお, JVMTI 仕様により, 対象のスレッドは suspend していなければいけないと決められている).

実際の強制復帰処理は, JvmtiEnv::ForceEarlyReturn*() 内では行わず, 
deopt 処理または suspend 状態から通常状態に復帰するパスの途中で行われる

この処理は以下の流れで行われる.

1. まず, JvmtiEnv::PopFrame() 内でいくつかのフィールドがセットされる
   (なお対象のフレームが compiled frame の場合は deopt も行われる).

2. 上記のフィールドがセットされていると, 
   deopt 処理または suspend 状態から通常状態に復帰する処理において, 
   Interpreter::remove_activation_early_entry() が指すコードにジャンプする.

3. Interpreter::remove_activation_early_entry() 内で実際の強制復帰処理が行われる.

### 1. JvmtiEnv::ForceEarlyReturn*() 内での処理
セットされるフィールドは以下の通り.

  * JvmtiThreadState::_earlyret_state フィールド
  * JvmtiThreadState::_earlyret_oop フィールド
  * JvmtiThreadState::_earlyret_tos フィールド
  * JvmtiThreadState::_earlyret_value フィールド
  * JvmtiThreadState::_pending_step_for_earlyret フィールド


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      // JVMTI ForceEarlyReturn support
    
      // This is set to earlyret_pending to signal that top Java frame
      // should be returned immediately
     public:
      int           _earlyret_state;
      TosState      _earlyret_tos;
      jvalue        _earlyret_value;
      oop           _earlyret_oop;         // Used to return an oop result into Java code from
```


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      bool              _pending_step_for_earlyret;
```

### 2. deopt 処理または suspend 状態から通常状態に復帰する処理
ジャンプするまでの流れは以下の通り.

  * 処理対象が compiled frame の場合:

    JvmtiEnv::ForceEarlyReturn*() 内で対象のフレームが deopt される
    (正確には, 対象のスレッドが実行を再会し, そのフレームにリターンした際に deopt が走る).
    この deopt 処理の中で, JvmtiThreadState::_earlyret_state フィールドが参照されている.
    フィールドがセットされている場合, deopt 後の復帰先は
    Interpreter::remove_activation_early_entry() が返すアドレスになる.

  * 処理対象が template interpreter frame の場合:

    suspend する関数呼び出しからの復帰パスで処理が行われる.
    具体的には call_VM_base() による VM 呼び出しからのリターン後の処理で
    (InterpreterMacroAssembler::check_and_handle_popframe() で生成されたコードにより)
    JvmtiThreadState::_earlyret_state フィールドがチェックされ,
    Interpreter::remove_activation_early_entry() が返すアドレスにジャンプする.

### 3. Interpreter::remove_activation_preserving_args_entry() 内の処理
処理は以下のようになる.



なお, SingleStep モードで実行している際には, さらに SingleStep 用の後片付け処理も行われる.


## 備考(Notes)
Interpreter::remove_activation_early_entry() が指すコードは
TemplateInterpreterGenerator::generate_earlyret_entry_for() により生成されている.

なお, 現状では, C++ Interpreter 使用時には
(onload_capabilities に can_force_early_return が含まれないので) ForceEarlyReturn*() は使用できない模様.

#### 参考: JvmtiManageCapabilities::init_onload_capabilities()
See: [here](no2935oMn.html) for details
## 備考(Notes)
内部処理的には, ForceEarlyReturn*() は PopFrame() とかなり似ている (See: [here](no2935cDo.html) for details).

PopFrame に似てて, 最後の処理だけ違う, とのこと.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEnvBase.cpp))
    // ForceEarlyReturn<type> follows the PopFrame approach in many aspects.
    // Main difference is on the last stage in the interpreter.
    // The PopFrame stops method execution to continue execution
    // from the same method call instruction.
    // The ForceEarlyReturn forces return from method so the execution
    // continues at the bytecode following the method call.
```


## 処理の流れ (概要)(Execution Flows : Summary)
### ForceEarlyReturn*() の処理
```
JvmtiEnv::ForceEarlyReturn*()
-> JvmtiEnvBase::force_early_return()
   -> (1) 先頭フレームが compiled frame であれば interpreter frame に戻しておく.
          -> JvmtiEnvBase::check_top_frame()

      (2) ForceEarlyReturn* の処理対象になっていることを JvmtiThreadState オブジェクト内に記録しておく
          -> JvmtiThreadState::set_earlyret_pending()
          -> JvmtiThreadState::set_earlyret_oop()
          -> JvmtiThreadState::set_earlyret_value()
          -> JvmtiThreadState::set_pending_step_for_earlyret()
```

### 実際の強制復帰処理 
#### compiled frame の場合
```
deopt 処理 (See: )
-> vframeArrayElement::unpack_on_stack()
   -> JvmtiThreadState::is_earlyret_pending() が true の場合は,
      復帰先の PC として Interpreter::remove_activation_early_entry() を選択する
      (なお, JvmtiThreadState::is_earlyret_pending() は
      JvmtiThreadState::_earlyret_state フィールドをチェックする関数)
```

#### template interpreter frame の場合
```
* sparc の場合:

  (略) (See: [here](no2935dSX.html) for details)
  -> MacroAssembler::call_VM_base() が生成するコード
     -> MacroAssembler::check_and_forward_exception() が生成するコード
        -> InterpreterMacroAssembler::check_and_handle_earlyret() が生成するコード
           -> Interpreter::remove_activation_early_entry() が指しているコード
              (= TemplateInterpreterGenerator::generate_earlyret_entry_for() が生成するコード)

* x86 の場合:

  (略) (See: [here](no2935dSX.html) for details)
  -> MacroAssembler::call_VM_base() が生成するコード
     -> InterpreterMacroAssembler::check_and_handle_earlyret() が生成するコード
        -> Interpreter::remove_activation_early_entry() が指しているコード
           (= TemplateInterpreterGenerator::generate_earlyret_entry_for() が生成するコード)
```

### SingleStep 時の後片付け処理
```
(略) (See: [here](no7882EDP.html) for details)
-> JvmtiExport::at_single_stepping_point()
   -> JvmtiThreadState::is_pending_step_for_earlyret() が true の場合は,
      JvmtiThreadState::process_pending_step_for_earlyret() を呼び出す.
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::ForceEarlyReturnObject()
See: [here](no2935QBd.html) for details
### JvmtiEnv::ForceEarlyReturnInt()
See: [here](no2935dLj.html) for details
### JvmtiEnv::ForceEarlyReturnLong()
See: [here](no2935qVp.html) for details
### JvmtiEnv::ForceEarlyReturnFloat()
See: [here](no29353fv.html) for details
### JvmtiEnv::ForceEarlyReturnDouble()
See: [here](no2935Eq1.html) for details
### JvmtiEnv::ForceEarlyReturnVoid()
See: [here](no29352zE.html) for details
### JvmtiEnvBase::force_early_return()
See: [here](no2935D-K.html) for details
### JvmtiEnvBase::check_top_frame()
See: [here](no2935QIR.html) for details

### vframeArrayElement::unpack_on_stack()
See: [here](no2935pbW.html) for details

### MacroAssembler::check_and_forward_exception() (sparc の場合)
See: [here](no3059H0a.html) for details
### InterpreterMacroAssembler::check_and_handle_earlyret() (sparc の場合)
See: [here](no2935qcd.html) for details
### InterpreterMacroAssembler::check_and_handle_earlyret() (x86_64 の場合)
See: [here](no29353mj.html) for details

### TemplateInterpreterGenerator::generate_earlyret_entry_for() (sparc の場合)
See: [here](no2935Exp.html) for details
### TemplateInterpreterGenerator::generate_earlyret_entry_for() (x86_64 の場合)
See: [here](no2935R7v.html) for details






