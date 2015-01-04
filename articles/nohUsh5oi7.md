---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理(Loading)
---
[Up](no7882ALm.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理(Loading)

--- 
## 概要(Summary)
クラスのロード処理は SystemDictionary クラス内に実装されている.

クラスのロード処理は, HotSpot 内でクラス名に基づきクラス(klassOop)を取得しようとする際に発生する.
HotSpot 内では, SystemDictionary クラスによってクラス名とクラス(klassOop)の対応を管理しており, 
必要に応じて SystemDictionary からクラス(klassOop)を取得する (See: [here](no7ggAHQj6.html) for details).

SystemDictionary クラスは,
まだロードしていないクラス (SystemDictionary 内に存在しないクラス) が要求された場合, 
実際のロード処理を実行する (See: [here](noIvSV0NZj.html) for details).

ロード処理中では, クラスファイルのパース処理が実行され, 
これにより実際のクラス(klassOop)が生成される (See: [here](no2114rPX.html) for details).

## 参考(for your information)
* [HotSpot Runtime Overview : VM Class Loading](http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html#VM%20Class%20Loading|outline)
  



## Subcategories
* [Class のロード/リンク/初期化 ： ロード処理 (1) ： ロード処理の開始点](no7ggAHQj6.html)
* [Class のロード/リンク/初期化 ： ロード処理 (2) ： ロード処理の流れ (= SystemDictionary クラスによる resolve 処理)](noIvSV0NZj.html)
* [Class のロード/リンク/初期化 ： ロード処理 (3) ： クラスファイルのパース処理 ](no2114rPX.html)



