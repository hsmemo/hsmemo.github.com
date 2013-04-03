---
layout: default
title: Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： PopFrame() の処理  
---
[Up](noa-wMJL5x.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI の処理 ： JVMTI 関数の処理 ： スタックフレーム (Stack Frame) ： PopFrame() の処理  

--- 
## 概要(Summary)
(See: JVMTI 仕様)
(なお, JVMTI 仕様により, 対象のスレッドは suspend していなければいけないと決められている).

実際のフレームの pop 処理は, JvmtiEnv::PopFrame() 内では行わず, 
deopt 処理または suspend 状態から通常状態に復帰するパスの途中で行われる

この処理は以下の流れで行われる.

1. まず, JvmtiEnv::PopFrame() 内でいくつかのフィールドがセットされる
   (なお対象のフレームが compiled frame の場合は deopt も行われる).

2. 上記のフィールドがセットされていると, 
   deopt 処理または suspend 状態から通常状態に復帰する処理において, 
   Interpreter::remove_activation_preserving_args_entry() が指すコードにジャンプする.

3. Interpreter::remove_activation_preserving_args_entry() 内で実際のフレームの pop 処理が行われる.

### 1. JvmtiEnv::PopFrame() 内での処理
セットされるフィールドは以下の通り.

  * JavaThread::_popframe_condition フィールド
 
    (ほとんどの PopFrame() の処理はこちらのフィールドを参照して行われる)


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      // JVMTI PopFrame support
      // This is set to popframe_pending to signal that top Java frame should be popped immediately
      int _popframe_condition;
```

  * JvmtiThreadState::_pending_step_for_popframe フィールド
 
    (こちらは SingleStep モードにおける追加処理用. (See: [here](no7882EDP.html) for details))


```
    ((cite: hotspot/src/share/vm/prims/jvmtiThreadState.hpp))
      bool              _pending_step_for_popframe;
```

### 2. deopt 処理または suspend 状態から通常状態に復帰する処理
ジャンプするまでの流れは以下の通り.

  * 処理対象が compiled frame の場合:
    
    JvmtiEnv::PopFrame() 内で対象のフレームが deopt される
    (正確には, 対象のスレッドが実行を再会し, そのフレームにリターンした際に deopt が走る).
    この deopt 処理の中で, JavaThread::_popframe_condition フィールドが参照されている.
    フィールドがセットされている場合, deopt 後の復帰先は
    Interpreter::remove_activation_preserving_args_entry() が返すアドレスになる.
 
  * 処理対象が template interpreter frame の場合:
    
    suspend する関数呼び出しからの復帰パスで処理が行われる.
    具体的には call_VM_base() による VM 呼び出しからのリターン後の処理で
    (InterpreterMacroAssembler::check_and_handle_popframe() で生成されたコードにより)
    JavaThread::_popframe_condition フィールドがチェックされ,
    Interpreter::remove_activation_preserving_args_entry() が返すアドレスにジャンプする.
  
### 3. Interpreter::remove_activation_preserving_args_entry() 内の処理
処理は以下のようになる.

  * 処理対象のフレームの呼び出し元が template interpreter frame の場合:
    
    フレームを破棄した後, dispatch_next() で invoke 命令用のテンプレートに再度ジャンプする.
 
  * 処理対象のフレームの呼び出し元が compiled frame の場合:
    
    すぐに invoke を再実行するわけにはいかないので,
    一回 deopt で caller を interpreter frame に変換した後, 引数を積み直して再実行する
    (なお, caller の deopt 自体は JvmtiEnv::PopFrame() によって既に予約されており, それに乗っかるような形になる).
    
    (この場合には, Interpreter::remove_activation_preserving_args_entry() 内で
    Deoptimization::popframe_preserve_args() という関数を呼ばれ, 詰め直すべき引数が一時的に退避される.
    JavaThread::_popframe_condition フィールドに
    JavaThread::popframe_force_deopt_reexecution_bit というビットが立っていればこの処理中.
    このビットが立っていると, 退避した引数が deopt 処理内でオペランドスタックに詰めなおされ, 再度メソッド呼び出しが可能な状態が整う.
    
    (See: JavaThread::popframe_forcing_deopt_reexecution())
    (See: Deoptimization::fetch_unroll_info_helper())
    (See: vframeArrayElement::unpack_on_stack()))

なお, SingleStep モードで実行している際には, さらに SingleStep 用の後片付け処理も行われる.


## 備考(Notes)
Interpreter::remove_activation_preserving_args_entry() が指すコードは
TemplateInterpreterGenerator::generate_throw_exception() 内でコード生成されている.

なお, 現状では C++ Interpreter 使用時には
(onload_capabilities に can_pop_frame が含まれないので) PopFrame() は使用できない模様.

### 参考: JvmtiManageCapabilities::init_onload_capabilities()
See: [here](no2935oMn.html) for details
## 備考(Notes)
内部処理的には, PopFrame() は ForceEarlyReturn*() とかなり似ている (See: [here](no3059azN.html) for details).


## 処理の流れ (概要)(Execution Flows : Summary)
### PopFrame() の処理
```
JvmtiEnv::PopFrame()
-> (1) 先頭2つのフレームのうち compiled frame のものについては interpreter frame に戻しておく.
       -> Deoptimization::deoptimize_frame()

   (2) 先頭フレームに対して FramePop イベント通知が登録されていれば取り消す
       -> JvmtiThreadState::update_for_pop_top_frame()

   (3) PopFrame の処理対象になっていることを JavaThread オブジェクトや JvmtiThreadState オブジェクト内に記録しておく
       -> JavaThread::set_popframe_condition()
       -> JvmtiThreadState::set_pending_step_for_popframe()
```

### 実際の pop 処理 
#### compiled frame の場合
```
deopt 処理 (See: )
-> vframeArrayElement::unpack_on_stack()
   -> JavaThread::has_pending_popframe() が true の場合は,
      復帰先の PC として Interpreter::remove_activation_preserving_args_entry() を選択する
      (なお, JavaThread::has_pending_popframe() は
      JavaThread::_popframe_condition フィールドをチェックする関数)
```

#### template interpreter frame の場合
```
* sparc の場合:

  (略) (See: [here](no2935dSX.html) for details)
  -> MacroAssembler::call_VM_base() が生成するコード
     -> MacroAssembler::check_and_forward_exception() が生成するコード
        -> InterpreterMacroAssembler::check_and_handle_popframe() が生成するコード
           -> Interpreter::remove_activation_preserving_args_entry() が指しているコード
              (= TemplateInterpreterGenerator::generate_throw_exception() が生成するコード)

* x86 の場合:

  (略) (See: [here](no2935dSX.html) for details)
  -> MacroAssembler::call_VM_base() が生成するコード
     -> InterpreterMacroAssembler::check_and_handle_popframe() が生成するコード
        -> Interpreter::remove_activation_preserving_args_entry() が指しているコード
           (= TemplateInterpreterGenerator::generate_throw_exception() が生成するコード)
```

### SingleStep 時の後片付け処理
```
(略) (See: [here](no7882EDP.html) for details)
-> JvmtiExport::at_single_stepping_point()
   -> JvmtiThreadState::is_pending_step_for_popframe() が true の場合は,
      JvmtiThreadState::process_pending_step_for_popframe() を呼び出す
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JvmtiEnv::PopFrame()
See: [here](no2935P5h.html) for details
### Deoptimization::deoptimize_frame()
(#Under Construction)

### JvmtiThreadState::update_for_pop_top_frame()
See: [here](no2935pNu.html) for details
### JavaThread::set_popframe_condition()
See: [here](no29352X0.html) for details
### JvmtiThreadState::set_pending_step_for_popframe()
See: [here](no2935ohD.html) for details

### vframeArrayElement::unpack_on_stack()
See: [here](no2935pbW.html) for details
### AbstractInterpreter::layout_activation() (sparc の場合 & Template Interpreter の場合)
See: [here](no29352lc.html) for details
### AbstractInterpreter::layout_activation() (x86_64 の場合 & Template Interpreter の場合)
See: [here](no2935Dwi.html) for details

### MacroAssembler::check_and_forward_exception() (sparc の場合)
See: [here](no3059H0a.html) for details
### InterpreterMacroAssembler::check_and_handle_popframe() (sparc の場合)
See: [here](no29351rJ.html) for details
### TemplateInterpreterGenerator::generate_throw_exception() (sparc の場合)
See: [here](no3059OxC.html) for details
### InterpreterMacroAssembler::check_and_handle_popframe() (x86_64 の場合)
See: [here](no2935C2P.html) for details
### TemplateInterpreterGenerator::generate_throw_exception() (x86_64 の場合)
See: [here](no3059b7I.html) for details

### JvmtiThreadState::is_pending_step_for_popframe()
See: [here](no2935Dpu.html) for details
### JvmtiThreadState::process_pending_step_for_popframe()
(#Under Construction)
See: [here](no2935Qz0.html) for details







