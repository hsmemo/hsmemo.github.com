---
layout: default
title: Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (5) ： 呼び出し元(caller 側)での return 後の後片付け処理 ： Template Interpreter の場合  
---
[Up](noLA4BL5jt.html) [Top](../index.html)

#### Method に関する処理 ： Java のコードによるメソッド呼び出し処理 (5) ： 呼び出し元(caller 側)での return 後の後片付け処理 ： Template Interpreter の場合  

--- 
## 概要(Summary)
Template Interpreter の場合,
TemplateInterpreterGenerator::generate_return_entry_for() が生成したコードによって後片付けが行われる.

これは, 各invoke*命令が実行された際に, 呼び出し先からのリターンアドレスとして
(現在の PC の代わりに) 上記のreturn_entry()が生成したコードの先頭アドレスを設定することで実現している (See: [here](nolikc3vKY.html) for details).
このため, 呼び出し先から return してくると, 本当の呼び出し元である invoke*処理用の codelet の中ではなく, return entry の先頭に飛ぶことになる.

TemplateInterpreterGenerator::generate_return_entry_for() が生成したコード内では
「次の命令 (bcp の次の1byte) を調べて適切な codelet にジャンプする」という処理が行われる.
このため, 呼び出された側は普通にリターンするだけでいい.
リターンによって return_entry() が生成したスタブに飛び, そこで次の命令を調べて適切な codelet への遷移が起こる.

## 備考(Notes)
invoke* の種類によって次の命令の位置が変わるため (3byte 先 or 5 byte 先), 
TemplateInterpreter::return_entry() では両方のパターンを作っておき,
prepare_invoke() 等の中で引数に応じて適切なアドレスを選んでいる (See: [here](nolikc3vKY.html) for details).


## 処理の流れ (概要)(Execution Flows : Summary)
### 後片付け処理コードの生成処理
<div class="flow-abst"><pre>
(See: <a href="no3059kZk.html">here</a> for details)
-&gt; TemplateInterpreterGenerator::generate_all()
   -&gt; TemplateInterpreterGenerator::generate_return_entry_for()
</pre></div>

### 後片付け処理
<div class="flow-abst"><pre>
TemplateInterpreterGenerator::generate_return_entry_for() が生成したコード
-&gt; (1) レジスタの値を復帰させる
   (1) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成するコード
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateInterpreterGenerator::generate_return_entry_for() (sparc の場合)
See: [here](no3059gHg.html) for details
### TemplateInterpreterGenerator::generate_return_entry_for() (x86_64 の場合)
See: [here](no3059tRm.html) for details






