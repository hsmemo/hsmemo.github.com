---
layout: default
title: JIT Compiler の処理 (3) ： JIT コンパイラによるコード生成処理 ： ネイティブメソッドではない場合
---
[Up](no-wa6OPxh.html) [Top](../index.html)

#### JIT Compiler の処理 (3) ： JIT コンパイラによるコード生成処理 ： ネイティブメソッドではない場合

--- 
## 概要(Summary)
JIT コンパイラによるコードの生成処理は, 対象がネイティブメソッドではない場合,
AbstractCompiler::compile_method() で実行される (See: [here](nobV3Ayv16.html) for details).

ただし, AbstractCompiler::compile_method() は(事実上)仮想関数であり,
実際には使用する JIT コンパイラ種別に応じた各サブクラスの関数が呼ばれる.
実際に呼び出される関数は以下の通り.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table1098135C -->
| Compiler type | Function |
|---|---|
| C1 JIT コンパイラ | Compiler::compile_method() |
| C2 JIT コンパイラ | C2Compiler::compile_method() |
| Shark JIT コンパイラ | SharkCompiler::compile_method() |
<!-- END RECEIVE ORGTBL table1098135C -->

<!-- 
#+ORGTBL: SEND table1098135C orgtbl-to-gfm :no-escape t
| Compiler type        | Function                        |
|----------------------+---------------------------------|
| C1 JIT コンパイラ    | Compiler::compile_method()      |
| C2 JIT コンパイラ    | C2Compiler::compile_method()    |
| Shark JIT コンパイラ | SharkCompiler::compile_method() |
-->




## Subcategories
* [(#TBD) JIT Compiler の処理 (3) ： JIT コンパイラによるコード生成処理 ： ネイティブメソッドではない場合 ： C1 の場合](noWKg0YPu8.html)
* [(#TBD) JIT Compiler の処理 (3) ： JIT コンパイラによるコード生成処理 ： ネイティブメソッドではない場合 ： C2 の場合](noo7BHeg-E.html)
* [(#TBD) JIT Compiler の処理 (3) ： JIT コンパイラによるコード生成処理 ： ネイティブメソッドではない場合 ： Shark の場合](noiIylqdKX.html)



