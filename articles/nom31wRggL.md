---
layout: default
title: PhaseChaitin クラス関連のクラス (LRG, LRG_List, PhaseIFG, PhaseChaitin)
---
[Top](../index.html)

#### PhaseChaitin クラス関連のクラス (LRG, LRG_List, PhaseIFG, PhaseChaitin)

これらは, C2 JIT Compiler 内の処理フェーズを表すクラス.
より具体的に言うと, レジスタ割り当て処理を表すクラス.


### クラス一覧(class list)

  * [PhaseChaitin](#no36HjdHBU)
  * [LRG](#nolDTFW2P_)
  * [LRG_List](#noyUCzgRdv)
  * [PhaseIFG](#novNTzNFQc)


---
## <a name="no36HjdHBU" id="no36HjdHBU">PhaseChaitin</a>

### 概要(Summary)
PhaseRegAlloc クラスの具象サブクラス.

このクラスは, Chaitin (Briggs-Chaitin) の graph coloring によるレジスタ割り当てを行う.


```cpp
    ((cite: hotspot/src/share/vm/opto/chaitin.hpp))
    //------------------------------Chaitin----------------------------------------
    // Briggs-Chaitin style allocation, mostly.
    class PhaseChaitin : public PhaseRegAlloc {
```

### 使われ方(Usage)
Compile::Code_Gen() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseChaitin.html) for details

---
## <a name="nolDTFW2P_" id="nolDTFW2P_">LRG</a>

### 概要(Summary)
PhaseChaitin クラス用の補助クラス.

live-range を表すためのクラス. 
なおクラス名は "Live-RanGe" の略とのこと.


```cpp
    ((cite: hotspot/src/share/vm/opto/chaitin.hpp))
    //------------------------------LRG--------------------------------------------
    // Live-RanGe structure.
    class LRG : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 PhaseIFG オブジェクトの _lrgs フィールドに(のみ)格納されている.
 
(正確には, このフィールドは LRG の配列を格納するフィールド.
その PhaseIFG オブジェクト内で使用される LRG オブジェクトは全てこの中に格納されている)

#### 生成箇所(where its instances are created)
配列用のメモリ領域は PhaseIFG::init() 内で(のみ)確保されている. 




### 詳細(Details)
See: [here](../doxygen/classLRG.html) for details

---
## <a name="noyUCzgRdv" id="noyUCzgRdv">LRG_List</a>

### 概要(Summary)
PhaseChaitin クラス用の補助クラス.

各 Node と LRG の対応付けを管理するためのクラス
(より正確には, Node の index から LRG への写像).


```cpp
    ((cite: hotspot/src/share/vm/opto/chaitin.hpp))
    //------------------------------LRG_List---------------------------------------
    // Map Node indices to Live RanGe indices.
    // Array lookup in the optimized case.
    class LRG_List : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 PhaseChaitin オブジェクトの _names フィールド
* 各 PhaseChaitin オブジェクトの _uf_map フィールド

#### 生成箇所(where its instances are created)
(上記のフィールドは全て, ポインタ型ではなく実体なので,
 格納しているオブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classLRG__List.html) for details

---
## <a name="novNTzNFQc" id="novNTzNFQc">PhaseIFG</a>

### 概要(Summary)
PhaseChaitin クラス内で使用される補助クラス.

Phase クラスの具象サブクラスの1つ.
干渉グラフ(interference graph)の作成を行う.


```cpp
    ((cite: hotspot/src/share/vm/opto/chaitin.hpp))
    //------------------------------IFG--------------------------------------------
    //                         InterFerence Graph
    // An undirected graph implementation.  Created with a fixed number of
    // vertices.  Edges can be added & tested.  Vertices can be removed, then
    // added back later with all edges intact.  Can add edges between one vertex
    // and a list of other vertices.  Can union vertices (and their edges)
    // together.  The IFG needs to be really really fast, and also fairly
    // abstract!  It needs abstraction so I can fiddle with the implementation to
    // get even more speed.
    class PhaseIFG : public Phase {
```

### 使われ方(Usage)
PhaseChaitin::Register_Allocate() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPhaseIFG.html) for details

---
