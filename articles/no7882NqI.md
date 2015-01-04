---
layout: default
title: 実行時コンスタントプールのシンボルの解決処理(resolution)  
---
[Up](no38NSe1ks.html) [Top](../index.html)

#### 実行時コンスタントプールのシンボルの解決処理(resolution)  

--- 
## 概要(Summary)
実行中に Constant Pool 中の各シンボルが参照されると,
その具体的な値を探す処理(解決処理)が行われる (See: [here](noDbg01tAk.html) for details).

実際の解決処理は constantPoolOopDesc クラス及び LinkResolver クラスに実装されている (#TODO)
(See: [here](no-VW6a46T.html) for details).

## 参考(for your information)
* Java 仮想マシン仕様 (5.4.3 "Resolution")

## 備考(Notes)
* クラス名/インタフェース名の解決処理では, 処理の高速化のため,
  初回に解決した結果を constantPoolOopDesc 内にキャッシュしている.
  また文字列オブジェクトの解決時にも同様にキャッシュしている
  (See: [here](no-VW6a46T.html) for details).

* インタープリタでは, フィールド名やメソッド名の解決時処理を高速化するために, 
  constantPoolCacheOop オブジェクトを用いて一度行った解決結果をキャッシュしている
  (See: [here](noX7cqxfNI.html) for details).




## Subcategories
* [実行時コンスタントプールのシンボルの解決処理(resolution) (1) ： 解決処理の開始契機](noDbg01tAk.html)
* [実行時コンスタントプールのシンボルの解決処理(resolution) (2) ： 解決処理](no-VW6a46T.html)



