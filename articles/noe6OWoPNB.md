---
layout: default
title: Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： ランタイム処理中に検出される例外
---
[Up](noGGgR6UDC.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： ランタイム処理中に検出される例外

--- 
#Under Construction

## 概要(Summary)
ランタイムの処理中に例外送出条件が満たされると, その時点で例外送出処理が開始される.

ランタイム処理中に検出される例外は以下の通り.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table6801U4r -->
| Exception | Description |
|---|---|
| NegativeArraySizeException | anewarray でサイズがマイナスの配列を作ろうとした場合 |
| IllegalMonitorStateException | monitorexit や synchronized method の return 時 |
| IncompatibleClassChangeError | getstatic 等でダイナミックリンクが走った場合 |
| OutOfMemoryError | new 等でのメモリ確保時に, 空きメモリの不足により失敗した場合 |
| NoSuchMethodError, AbstractMethodError, LinkageError, ... | ダイナミックリンク処理が失敗した場合 |
| ... (#TODO) |  |
<!-- END RECEIVE ORGTBL table6801U4r -->

<!-- 
#+ORGTBL: SEND table6801U4r orgtbl-to-gfm :no-escape t
| Exception                                                 | Description                                                  |
|-----------------------------------------------------------+--------------------------------------------------------------|
| NegativeArraySizeException                                | anewarray でサイズがマイナスの配列を作ろうとした場合         |
| IllegalMonitorStateException                              | monitorexit や synchronized method の return 時              |
| IncompatibleClassChangeError                              | getstatic 等でダイナミックリンクが走った場合                 |
| OutOfMemoryError                                          | new 等でのメモリ確保時に, 空きメモリの不足により失敗した場合 |
| NoSuchMethodError, AbstractMethodError, LinkageError, ... | ダイナミックリンク処理が失敗した場合                         |
| ... (#TODO)                                               |                                                              |
-->

## 処理の流れ (概要)(Execution Flows : Summary)
(#Under Construction)






