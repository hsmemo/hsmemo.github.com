---
layout: default
title: Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： 非ランタイム(かつネイティブ)フレーム → ランタイムフレームの境界での処理
---
[Up](noAJsAY6Zl.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： 非ランタイム(かつネイティブ)フレーム → ランタイムフレームの境界での処理

--- 
## 概要(Summary)
特に処理はない
(ネイティブメソッド内では明示的に例外をチェックするため, 
pending_exception に値がセットされているだけでよい) (See: [here](nok1NPdCrM.html) for details)

## 備考(Notes)
より正確に言うと, この境界では JNI_ENTRY() および JNI_END() マクロによるチェックが行われるが, 
例外については特に処理されていない (See: [here](no6897-YH.html) for details).







