---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (3) TLAB の確保処理
---
[Up](no30267vB.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (3) TLAB の確保処理

--- 
## 概要(Summary)
TLAB の確保処理は CollectedHeap::allocate_new_tlab() メソッドで行われる (See: [here](no28916Q0G.html) for details).
ただし, 実際にはこのメソッドは各サブクラスでオーバーライドされているため, それぞれのサブクラス毎 (= GC アルゴリズム毎) の処理が呼び出される.

* ParallelScavengeHeap の場合: 

  (See: [here](no28916rXC.html) for details)

* G1collectedheap の場合:
  
  (See: [here](no289164hI.html) for details)

* GenCollectedHeap の場合:
  
  (See: [here](no28916FsO.html) for details)



## Subcategories
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (3) TLAB の確保処理 ： ParallelScavengeHeap の場合](no28916rXC.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (3) TLAB の確保処理 ： G1collectedheap の場合](no289164hI.html)
* [Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (3) TLAB の確保処理 ： GenCollectedHeap の場合](no28916FsO.html)



