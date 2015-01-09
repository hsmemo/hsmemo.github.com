---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理
---
[Up](nohhZ5yDpa.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理

--- 
## 概要(Summary)
InvocationCounter のカウンタ値が閾値を超えると,
CompilationPolicy オブジェクトの CompilationPolicy::event() が呼び出される (See: [here](nohhZ5yDpa.html) for details).

実際には CompilationPolicy オブジェクトには以下の4つのクラスがあり,
それぞれのクラスで CompilationPolicy::event() をオーバーライドした関数が実行される
(See: [here](no3420bIr.html), [here](no34200pY.html), [here](no3420O-k.html) and [here](no3420B0e.html) for details).

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table10981LSu -->
| Subclass | Function |
|---|---|
| SimpleCompPolicy | NonTieredCompPolicy::event() |
| StackWalkCompPolicy | NonTieredCompPolicy::event() |
| SimpleThresholdPolicy | SimpleThresholdPolicy::event() |
| AdvancedThresholdPolicy | SimpleThresholdPolicy::event() |
<!-- END RECEIVE ORGTBL table10981LSu -->

<!-- 
#+ORGTBL: SEND table10981LSu orgtbl-to-gfm :no-escape t
| Subclass                | Function                       |
|-------------------------+--------------------------------|
| SimpleCompPolicy        | NonTieredCompPolicy::event()   |
| StackWalkCompPolicy     | NonTieredCompPolicy::event()   |
| SimpleThresholdPolicy   | SimpleThresholdPolicy::event() |
| AdvancedThresholdPolicy | SimpleThresholdPolicy::event() |
-->

この関数で JIT コンパイルを行うべきと判断された場合, 
CompileBroker::compile_method() が呼び出されて JIT コンパイルが開始される.
(See: [here](nobV3Ayv16.html) for details)

## 備考(Notes)
* なお, CompilationPolicy オブジェクトは HotSpot の起動時に生成されている (See: [here](nopbZ_pxeK.html) for details).




## Subcategories
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： SimpleCompPolicy の場合 ](no3420bIr.html)
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： StackWalkCompPolicy の場合 ](no34200pY.html)
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： SimpleThresholdPolicy の場合 ](no3420O-k.html)
* [(#TBD) JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理 ： AdvancedThresholdPolicy の場合 ](no3420B0e.html)



