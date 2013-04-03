---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中での Safepoint 停止処理
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中での Safepoint 停止処理

--- 
## 概要(Summary)
ネイティブコード (JNI のネイティブメソッド等) の場合, 
HotSpot 内の制御とは無関係なのでメソッド実行中に強制的に停止させる方法はない.

このため, 処理が HotSpot 内に戻ってきたタイミングで明示的にチェックさせて停止させている.
具体的には以下の箇所が Safepoint になる.

* ネイティブコードから HotSpot 内へのリターン時
* ネイティブコードによる JNI 関数の呼び出し時
* ネイティブコードによる CVMI 関数の呼び出し時

(<= 逆に言うと HotSpot 内に戻ってこなければチェックもないので停止されないが, ネイティブコードについてはそれでよいという設計方針になっている.
このため, ネイティブコード実行中のスレッドに付いては VM Thread も停止するまで待たない.
どのみち HotSpot 内に戻ってこなければ VM Operation 等に支障を来すような処理は行えないので, 停止していなくても問題はない)




## Subcategories
* [Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： HotSpot 内へのリターン時](nog8dNLL1D.html)
* [Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： JNI Functions の呼び出し時 (JNI_*)  ](no6897-YH.html)
* [Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： CVMI 関数の呼び出し時 (JVM_*)](noiJ3pF9IP.html)



