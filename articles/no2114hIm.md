---
layout: default
title: Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (メモリ領域の確保, etc) 
---
[Up](no6897XsM.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (メモリ領域の確保, etc) 

--- 
## 概要(Summary)
メモリ関係のデータ構造は HotSpot の起動時に初期化される.
この初期化処理の流れは以下のようになる.

1. 使用する GC アルゴリズムを決定する
2. 決定した GC アルゴリズムに応じた CollectedHeap クラスを生成＆初期化する
3. GC アルゴリズムとして CMS か G1GC が指定されている場合には SurrogateLockerThread も生成する

## 備考(Notes)
実際の仮想メモリ空間の確保処理(reserve 処理)は, ReservedSpace のコンストラクタで os::reserve_memory() により行われる.
その後, 適当な箇所(各Generationのコンストラクタ内等)で os::commit_memory() が呼び出されてコミットされる
(See: [here](noS8y7MAwP.html) for details).




## Subcategories
* [Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (1)  ](noYV_1Xq7P.html)
* [Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： CollectedHeap オブジェクトの初期化処理の流れ](noS8y7MAwP.html)
* [Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (3) ： SurrogateLockerThread の生成処理の流れ  ](no7882OK1.html)



