---
layout: default
title: Class のロード/リンク/初期化 ： 初期化処理(Initializing)
---
[Up](no7882ALm.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： 初期化処理(Initializing)

--- 
## 概要(Summary)
クラスの初期化処理は Klass::initialize() に実装されている.

HotSpot 内では, クラスをロードした後でも, 実際に使用されるまでは未初期化のままとしている.
このため, HotSpot 内でクラスの情報にアクセスする際(フィールドアクセス, メソッド呼び出し, インスタンス生成, etc)には, 
初期化されていない可能性を考慮し,
念のために Klass::initialize() を呼び出してから処理を行う
(See: [here](noYfLPYyAt.html) for details).

Klass::initialize() 内では, 対象のクラスがまだ初期化されていなければ, 初期化処理を行う
(See: [here](no9AAGw84F.html) for details).




## Subcategories
* [Class のロード/リンク/初期化 ： 初期化処理 (1) ： 初期化処理の開始点](noYfLPYyAt.html)
* [Class のロード/リンク/初期化 ： 初期化処理 (2) ： 初期化処理の流れ](no9AAGw84F.html)



