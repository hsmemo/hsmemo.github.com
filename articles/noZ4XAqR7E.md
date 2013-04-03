---
layout: default
title: Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (1) ： 呼び出し元(caller側)での invoke 処理 ： Template Interpreter の場合 ： invokespecial の処理
---
[Up](nolikc3vKY.html) [Top](../index.html)

#### Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (1) ： 呼び出し元(caller側)での invoke 処理 ： Template Interpreter の場合 ： invokespecial の処理

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
```
TemplateTable::invokespecial() が生成したコード
-> InterpreterMacroAssembler::call_from_interpreter()  が生成したコード
   -> (実際の呼び出し先にジャンプ)
```

### x86_64 の場合
```
TemplateTable::invokespecial() が生成したコード
-> InterpreterMacroAssembler::jump_from_interpreted()  が生成したコード
   -> InterpreterMacroAssembler::prepare_to_jump_from_interpreted()  が生成したコード
   -> (実際の呼び出し先にジャンプ)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::invokespecial()  (sparc の場合)
See: [here](no3059Sn2.html) for details
### TemplateTable::load_invoke_cp_cache_entry() (sparc の場合)
(#Under Construction)
See: [here](no3059ExF.html) for details
### InterpreterMacroAssembler::call_from_interpreter()
See: [here](no3059R7L.html) for details

### TemplateTable::invokespecial()  (x86_64 の場合)
See: [here](no3059RCA.html) for details
### TemplateTable::prepare_invoke()
See: [here](no3059Fkk.html) for details
### InterpreterMacroAssembler::save_bcp() (x86_64 の場合)
See: [here](no3059Suq.html) for details
### InterpreterMacroAssembler::jump_from_interpreted()
See: [here](no3059eMG.html) for details
### InterpreterMacroAssembler::prepare_to_jump_from_interpreted()
See: [here](no3059rWM.html) for details






