---
layout: default
title: Exception の処理 ： 処理の詳細 (3) ： 例外の送出処理 ： athrow の処理 ： Template Interpreter での処理
---
[Up](noHNONT0aT.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (3) ： 例外の送出処理 ： athrow の処理 ： Template Interpreter での処理

--- 
## 概要(Summary)
athrow の処理は, 基本的には TemplateInterpreterGenerator::generate_throw_exception() で生成した送出処理を実行するだけ.

## 処理の流れ (概要)(Execution Flows : Summary)
```
TemplateTable::athrow() が生成するコード
-> Interpreter::throw_exception_entry() が指しているコード (= TemplateInterpreterGenerator::generate_throw_exception() が生成したコード)
   -> (See: [here](no30593YX.html) for details)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::athrow() (sparc の場合)
See: [here](no3059CMz.html) for details
### TemplateTable::athrow() (x86_64 の場合)
See: [here](no30590VC.html) for details






