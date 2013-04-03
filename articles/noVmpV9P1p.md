---
layout: default
title: Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： 明示的に検出する例外 ： Template Interpreter の場合
---
[Up](nodVZs6as7.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： 明示的に検出する例外 ： Template Interpreter の場合

--- 
#Under Construction

## 概要(Summary)
例外を起こす可能性のある bytecode のテンプレートでは, 多くの場合, 明示的に例外条件をチェックしている.
例外送出条件が満たされていればランタイムを呼び出して例外を発生させる.

例外を起こすバイトコード命令は以下の通り.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table6801gWH -->
| Exception | Bytecodes |
|---|---|
| ArrayIndexOutOfBoundsException | 配列の load/store 系の命令全て |
| ArrayStoreException | 配列の store 系の命令全て |
| ClassCastException | checkcast |
| ... (#TODO) |  |
<!-- END RECEIVE ORGTBL table6801gWH -->

<!-- 
#+ORGTBL: SEND table6801gWH orgtbl-to-gfm :no-escape t
| Exception                      | Bytecodes                      |
|--------------------------------+--------------------------------|
| ArrayIndexOutOfBoundsException | 配列の load/store 系の命令全て |
| ArrayStoreException            | 配列の store 系の命令全て      |
| ClassCastException             | checkcast                      |
| ... (#TODO)                    |                                |
-->

## 処理の流れ (概要)(Execution Flows : Summary)
(#Under Construction)






