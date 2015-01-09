---
layout: default
title: JNI の処理 ： native method の処理 ： native method の呼び出し処理
---
[Up](noNisy_uNv.html) [Top](../index.html)

#### JNI の処理 ： native method の処理 ： native method の呼び出し処理

--- 
## 概要(Summary)
native method の呼び出し処理は「対象のメソッドが JIT コンパイルされたかどうか (= JIT 生成コードによる実行か Interpreter による実行か)」に応じて 2通りに分かれる.

どちらの場合も, 基本的には通常のメソッド呼び出し処理と変わらない (See: [here](nop2rLhpmY.html) for details).
ただし, 呼び出された側(callee 側)で行うメソッドの先頭部分での処理 (method entry 部での処理) がネイティブメソッド用の処理になる.

## 備考(Notes)
JVM 内では引数の受け渡しにオペランドスタック等を用いるが, ネイティブメソッドでは ABI に従ったレジスタ等を用いる必要がある.
また, Java メソッドと異なり, オペランドスタックの後ろにローカル変数領域を挿入する処理は要らない.

ネイティブメソッド用の method entry 部では,
HotSpot 内の calling convention にそった引数をネイティブの calling convention に合わせて詰め直し, 
その後に実際のネイティブコードを呼び出す処理が行われる.



## Subcategories
* [JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合  ](no3059asZ.html)
* [JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になった native method の場合  ](no1904s1R.html)



