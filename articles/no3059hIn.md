---
layout: default
title: 同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： JVMTI の PopFrame()/ForceEarlyReturn() 時 ： Template Interpreter での処理  
---
[Up](nom8Tnx-bW.html) [Top](../index.html)

#### 同期排他処理 ： ロック解放処理 ： fast-path の処理 ： synchronized method の exit 部 ： JVMTI の PopFrame()/ForceEarlyReturn() 時 ： Template Interpreter での処理  

--- 
## 概要(Summary)
処理は以下の関数が生成するコードによって行われる. また CPU 種別によって生成されるコードは異なる.

  * PopFrame() の場合: 
    
    TemplateInterpreterGenerator::generate_throw_exception() (See: [here](no2935cDo.html) for details)

  * ForceEarlyReturn() の場合: 
    
    TemplateInterpreterGenerator::generate_earlyret_entry_for() (See: [here](no3059azN.html) for details)

ただしどの場合も処理の流れは同じ.
具体的には, InterpreterMacroAssembler::unlock_object() が生成するコードでロック解放を行う.

解放が fast-path で終わらない場合は, InterpreterRuntime::monitorexit() による slow-path 処理にフォールバックする
(See: [here](noGAuAWXSd.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
### PopFrame 時の処理
#### sparc の場合
```
TemplateInterpreterGenerator::generate_throw_exception() が生成したコード (See: [here](no2935cDo.html) for details)
-> InterpreterMacroAssembler::unlock_if_synchronized_method() が生成したコード
   -> InterpreterMacroAssembler::unlock_object() が生成したコード
      -> MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -> InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -> (See: [here](noGAuAWXSd.html) for details)
```

#### x86_64 の場合
```
TemplateInterpreterGenerator::generate_throw_exception() が生成したコード (See: [here](no2935cDo.html) for details)
-> InterpreterMacroAssembler::remove_activation() が生成したコード (See: [here](no30590Am.html) for details)
   -> InterpreterMacroAssembler::unlock_object() が生成したコード
      -> MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -> InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -> (See: [here](noGAuAWXSd.html) for details)
```

### ForceEarlyReturn 時の処理
#### sparc の場合
```
TemplateInterpreterGenerator::generate_earlyret_entry_for() が生成したコード (See: [here](no3059azN.html) for details)
-> InterpreterMacroAssembler::remove_activation() が生成したコード (See: [here](no30590Am.html) for details)
   -> InterpreterMacroAssembler::unlock_if_synchronized_method() が生成したコード
      -> InterpreterMacroAssembler::unlock_object() が生成したコード
         -> MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
         -> InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
            -> (See: [here](noGAuAWXSd.html) for details)
```

#### x86_64 の場合
```
TemplateInterpreterGenerator::generate_earlyret_entry_for() が生成したコード (See: [here](no3059azN.html) for details)
-> InterpreterMacroAssembler::remove_activation() が生成したコード (See: [here](no30590Am.html) for details)
   -> InterpreterMacroAssembler::unlock_object() が生成したコード
      -> MacroAssembler::biased_locking_exit() が生成したコード  (← biased locking を使用している場合にのみ呼び出される)
      -> InterpreterRuntime::monitorexit()                    (← fast-path が成功しなかった場合にのみ呼び出す)
         -> (See: [here](noGAuAWXSd.html) for details)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterMacroAssembler::unlock_if_synchronized_method() (sparc の場合)
See: [here](no30590Ha.html) for details
### InterpreterMacroAssembler::unlock_object() (sparc の場合)
See: [here](no4230pQt.html) for details
### MacroAssembler::biased_locking_exit() (sparc の場合)
See: [here](no28916NCv.html) for details

### InterpreterMacroAssembler::unlock_object() (x86_64 の場合)
See: [here](no4230okC.html) for details
### MacroAssembler::biased_locking_exit() (x86_64 の場合)
See: [here](no28916MWE.html) for details






