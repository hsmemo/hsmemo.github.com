---
layout: default
title: Template Interpreter によるバイトコードの実行処理 ： 全テンプレートで共通な処理
---
[Up](noaqS079AL.html) [Top](../index.html)

#### Template Interpreter によるバイトコードの実行処理 ： 全テンプレートで共通な処理

--- 
## 概要(Summary)
どのテンプレートの場合も, 
テンプレートの先頭部分と末尾部分の処理は共通している.
これらの処理は以下の 2つの関数で生成される (See: [here](noZtCzUJGc.html) for details).

  * InterpreterMacroAssembler::dispatch_prolog() : テンプレートの先頭部分用のコード
  * InterpreterMacroAssembler::dispatch_epilog() : テンプレートの末尾部分用のコード (= 次のバイトコードにジャンプするコード)

それぞれのプラットフォームにおける処理の概要は以下の通り
(なお, IdispatchTables は dispatch table 自体のアドレスを入れておくレジスタ.
IdispatchAddress は dispatch table から取り出したアドレスを一時的に入れておくレジスタ).

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table9282g5W -->
|  | x86_64 | sparc |
|---|---|---|
| InterpreterMacroAssembler::dispatch_prolog() | 何もしない | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを取得し, IdispatchAddress に格納 |
| InterpreterMacroAssembler::dispatch_epilog() | dispatch_next() が生成するコードに丸投げしているだけ | IdispatchAddress にジャンプ |
| InterpreterMacroAssembler::dispatch_next() | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを生成してジャンプ | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを生成してジャンプ(※) |
<!-- END RECEIVE ORGTBL table9282g5W -->

<!-- 
#+ORGTBL: SEND table9282g5W orgtbl-to-gfm :no-escape t
|                                              | x86_64                                                                                   | sparc                                                                                                   |
|----------------------------------------------+------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------|
| InterpreterMacroAssembler::dispatch_prolog() | 何もしない                                                                               | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを取得し, IdispatchAddress に格納 |
| InterpreterMacroAssembler::dispatch_epilog() | dispatch_next() が生成するコードに丸投げしているだけ                                     | IdispatchAddress にジャンプ                                                                             |
| InterpreterMacroAssembler::dispatch_next()   | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを生成してジャンプ | 次のバイトコードと実行後の TosState を基に dispatch table からアドレスを生成してジャンプ(※)            |
-->

(※) (sparc 版では dispatch_next() は dispatch_epilog() 内では使われていないが,
     templateTable の branch や ret/wide_ret, tableswitch 等から使われている他, 
     generate_return_entry_for() 内などでも使われている)

## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterMacroAssembler::dispatch_prolog()  (sparc の場合)
See: [here](no7882vzl.html) for details
### InterpreterMacroAssembler::dispatch_prolog()  (x86_64 の場合)
See: [here](no788289r.html) for details
### InterpreterMacroAssembler::dispatch_epilog()  (sparc の場合)
See: [here](no78827RB.html) for details
### InterpreterMacroAssembler::dispatch_epilog()  (x86_64 の場合)
See: [here](no7882JIy.html) for details
### InterpreterMacroAssembler::dispatch_next()  (x86_64 の場合)
See: [here](no7882VmN.html) for details
### InterpreterMacroAssembler::dispatch_table()
See: [here](no7882iwT.html) for details
### InterpreterMacroAssembler::dispatch_base()
See: [here](no78828Eg.html) for details






