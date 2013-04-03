---
layout: default
title: Method に関する処理 ： calling convention (呼び出し規約) ： x86_64 の場合
---
[Up](noxTQWbUKc.html) [Top](../index.html)

#### Method に関する処理 ： calling convention (呼び出し規約) ： x86_64 の場合

--- 
## 概要(Summary)
System V の psABI とほぼ同様.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table28631ii -->
| Convention | Description |
|---|---|
| caller/callee register | System V psABI の通り(?? #TODO). |
| 引数 | 呼び出し先がインタープリタ実行される場合は, スタック上に積んで渡す (See: ...). JIT コンパイルされている場合は, レジスタで渡す (See: ...). |
| リターンアドレス | System V psABI の通り, スタックフレーム上の領域を使用. |
| 返値 | System V psABI の通り, RAX や XMM0 を使用. |
| SP のずれへの対応 | callee 側のエントリ処理や i2c/c2i stub により SP が勝手にずらされてしまった場合については, 全て interpreter 側で対処している(※) |
<!-- END RECEIVE ORGTBL table28631ii -->

<!-- 
#+ORGTBL: SEND table28631ii orgtbl-to-gfm :no-escape t
| Convention             | Description                                                                                                                               |
|------------------------+-------------------------------------------------------------------------------------------------------------------------------------------|
| caller/callee register | System V psABI の通り(?? #TODO).                                                                                                          |
| 引数                   | 呼び出し先がインタープリタ実行される場合は, スタック上に積んで渡す (See: ...). JIT コンパイルされている場合は, レジスタで渡す (See: ...). |
| リターンアドレス       | System V psABI の通り, スタックフレーム上の領域を使用.                                                                                    |
| 返値                   | System V psABI の通り, RAX や XMM0 を使用.                                                                                                |
| SP のずれへの対応      | callee 側のエントリ処理や i2c/c2i stub により SP が勝手にずらされてしまった場合については, 全て interpreter 側で対処している(※)          |
-->

(※) (interpreter 側の呼び出し時には return 後にずれを補正し,
      逆に interpreter が呼び出された時には, return 時にずれを考慮して SP の復帰処理を行う.
     これにより, compiled code 側には特別な処理は何も必要とされない)






