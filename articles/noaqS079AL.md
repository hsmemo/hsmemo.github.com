---
layout: default
title: Template Interpreter によるバイトコードの実行処理
---
[Up](noSYJTkIJn.html) [Top](../index.html)

#### Template Interpreter によるバイトコードの実行処理

--- 
## 概要(Summary)
Template Interpreter によるバイトコードの実行処理は, 
テンプレートと呼ばれるコード片によって実現される (See: ).
各バイトコード種別に対して対応するテンプレートが存在する. 

各テンプレートの生成は HotSpot の起動中に行われる (See: [here](noZtCzUJGc.html) for details).

また, テンプレートを構成するコードは2つの部分に大別できる.

  * 全てのテンプレートで共通な部分 (= Interpreter の実行制御に関する枠組み部分)  (See: [here](noEgWr8prQ.html) for details)
  * テンプレート毎に異なる部分 (= 各バイトコードの処理に対応する部分)  (See: )


## 備考(Notes)
バイトコードの分類は JVMS 2.11 に基づく. ただし, JVMS では以下の 4カテゴリーが1つにまとめられていたが, 説明の都合上ここでは4つに分割.

  * オブジェクト生成/配列生成 (new, newarray, anewarray, multianewarray)
  * フィールドアクセス (getfield, putfield, getstatic, putstatic)
  * 配列要素へのアクセス／配列の長さ取得 (?aload, ?astore, arraylength)
  * 動的型検査／型キャスト (instanceof, checkcast)



## Subcategories
* [Template Interpreter によるバイトコードの実行処理 ： テンプレートの生成処理](noZtCzUJGc.html)
* [Template Interpreter によるバイトコードの実行処理 ： 全テンプレートで共通な処理](noEgWr8prQ.html)
* [(#TBD) Template Interpreter によるバイトコードの実行処理 ： ロード命令／ストア命令 (?load, ?load_<n>, ?store, ?store_<n>, ?ipush, ldc*, ?const_*, wide)](nol1OD2rml.html)
* [Template Interpreter によるバイトコードの実行処理 ： 算術命令(算術演算, 論理演算, シフト, 比較) (?add, ?sub, ?mul, ?div, ?rem, ?neg, ?shl, ?shr, ?ushr, ?and, ?or, ?xor, iinc, ?cmp, ?cmpg, ?cmpl)](no6h-npuWN.html)
* [(#TBD) Template Interpreter によるバイトコードの実行処理 ： 型変換 (i2?, l2?, f2?, d2?)](nosMLy5xgr.html)
* [Template Interpreter によるバイトコードの実行処理 ： オブジェクト生成/配列生成 (new, newarray, anewarray, multianewarray)](noudJ3us0c.html)
* [Template Interpreter によるバイトコードの実行処理 ： フィールドアクセス (getfield, putfield, getstatic, putstatic)](noXBva0MBT.html)
* [(#TBD) Template Interpreter によるバイトコードの実行処理 ： 配列要素へのアクセス／配列の長さ取得 (?aload, ?astore, arraylength)](no5OWF9j69.html)
* [(#TBD) Template Interpreter によるバイトコードの実行処理 ： 動的型検査／型キャスト (instanceof, checkcast)](no5F7e42op.html)
* [(#TBD) Template Interpreter によるバイトコードの実行処理 ： オペランド・スタック管理 (pop*, dup*, swap)](noFjLJhfzP.html)
* [Template Interpreter によるバイトコードの実行処理 ： 制御の移行 (if*, tableswitch, lookupswitch, goto*, jsr*, ret)](noS59wryRf.html)
* [Template Interpreter によるバイトコードの実行処理 ： メソッド起動/リターン (invoke*, ?return/return)](noGGvnx01v.html)
* [Template Interpreter によるバイトコードの実行処理 ： 例外のスロー (athrow)](no8SJ62fep.html)
* [Template Interpreter によるバイトコードの実行処理 ： 同期化 (monitorenter, monitorexit)](noGFzgRIDH.html)



