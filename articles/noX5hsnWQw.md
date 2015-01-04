---
layout: default
title: Class のロード/リンク/初期化 ： リンク処理(Linking)
---
[Up](no7882ALm.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： リンク処理(Linking)

--- 
## 概要(Summary)
クラスのリンク処理は instanceKlass::link_class() (正確には instanceKlass::link_class_impl()) に実装されている.

HotSpot 内では, クラスをロードした後でも, 実際に使用されるまでは未リンク(＆未初期化)のままとしている.
このため, 必要になったタイミングで適宜リンク処理が開始される (See: [here](no8U-5iL83.html) for details)

instanceKlass::link_class() (instanceKlass::link_class_impl()) 内では,
対象のクラスがまだリンクされていなければ, リンク処理を行う
(See: [here](no3059xqe.html) for details).

リンク処理中では, バイトコードの verification 処理が行われる (See: [here](no7882amm.html) for details).
また, rewrite 処理 (高速化のために HotSpot 独自のバイトコードに置き換える処理) も行われる (See: [here](no3059AfB.html) for details).



## Subcategories
* [Class のロード/リンク/初期化 ： リンク処理 (1) ： リンク処理の開始点](no8U-5iL83.html)
* [Class のロード/リンク/初期化 ： リンク処理 (2) ： リンク処理の流れ  ](no3059xqe.html)
* [Class のロード/リンク/初期化 ： リンク処理 (3) ： バイトコードの検証処理(verification)  ](no7882amm.html)
* [Class のロード/リンク/初期化 ： リンク処理 (4) ： rewrite 処理 (& constant pool cache の作成) ](no3059AfB.html)



