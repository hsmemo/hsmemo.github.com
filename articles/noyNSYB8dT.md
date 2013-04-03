---
layout: default
title: Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： 非ランタイム(かつネイティブ)フレーム → 非ランタイム(かつ非ネイティブ)フレームの境界での処理
---
[Up](noAJsAY6Zl.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： 非ランタイム(かつネイティブ)フレーム → 非ランタイム(かつ非ネイティブ)フレームの境界での処理

--- 
## 概要(Summary)
特に処理はない
(ネイティブメソッド内では明示的に例外をチェックするため, 
pending_exception に値がセットされているだけでよい) (See: [here](nok1NPdCrM.html) for details)

## 備考(Notes)
より正確に言うと, 例外が起こった場合には, 
呼び出した JNI の Call*Method() 関数内で CHECK マクロによるリターンが繰り返され, 
最終的に Call*Method() 関数を呼び出したネイティブコードまでリターンするだけ (See: [here](no3059-0k.html) for details).







