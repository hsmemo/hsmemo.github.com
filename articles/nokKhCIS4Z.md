---
layout: default
title: VirtualSpace クラス関連のクラス (ReservedSpace, ReservedHeapSpace, ReservedCodeSpace, VirtualSpace)
---
[Top](../index.html)

#### VirtualSpace クラス関連のクラス (ReservedSpace, ReservedHeapSpace, ReservedCodeSpace, VirtualSpace)

これらは, メモリ管理用の補助クラス.
より具体的に言うと, 仮想メモリ空間の制御(領域の予約,領域のコミット)を行うためのユーティリティ・クラス
(要するに mmap() や VirtualAlloc() のラッパークラス) (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [ReservedSpace](#no0jxxJvRF)
  * [ReservedHeapSpace](#nomXLvzzes)
  * [ReservedCodeSpace](#no41wp5tEg)
  * [VirtualSpace](#noYBaoWzok)


---
## <a name="no0jxxJvRF" id="no0jxxJvRF">ReservedSpace</a>

### 概要(Summary)
指定したサイズの連続領域を仮想メモリ空間上に予約(reserve)するためのユーティリティ・クラス(ValueObjクラス).

(なお, ここでの「予約(reserve)」は
「仮想空間は確保したがアクセスすると segmentation fault になるかもしれない状態」を意味する. 
領域のコミット処理は VirtualSpace が行う. 
(まぁ Linux とかならあまり関係ないけど...))


```cpp
    ((cite: hotspot/src/share/vm/runtime/virtualspace.hpp))
    // ReservedSpace is a data structure for reserving a contiguous address range.
    
    class ReservedSpace VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. ReservedSpace 型の局所変数を宣言する (コンストラクタ引数で予約したいサイズを指定する)

   (なお, large page を使いたい場合や特定のアドレスから開始させたい場合にはそれも指定できる. ただし成功するとは限らないが...)

   (コンストラクタ内でメモリ空間の予約処理が行われる.
    なお, 生成した ReservedSpace オブジェクトの ReservedSpace.is_reserved() が false を返したら予約が失敗したことを意味する)

2. ReservedSpace クラスの各種のメソッドで, 予約できたアドレスに関する情報が取得できる 
   (ReservedSpace::base(), ReservedSpace::size()).

2. また ReservedSpace::first_part() メソッド及び ReservedSpace::last_part() メソッドで, 
   「予約したアドレス範囲中の前方から N バイトの領域」／「〃 前方 N バイトを除いた残りの領域」を表す
   ReservedSpace オブジェクトを生成することもできる.

#### 使用箇所(where its instances are used)
以下が, メモリ空間予約のために局所変数が生成されている箇所の一覧.

* CMSBitMap::allocate()
* CMSMarkStack::allocate()
* CMSMarkStack::expand()
* CMBitMapRO::CMBitMapRO()
* G1CollectedHeap::initialize()
* ObjectStartArray::initialize()
* ParMarkBitMap::initialize()
* ParallelCompactData::create_vspace()
* BlockOffsetSharedArray::BlockOffsetSharedArray()
* CardTableModRefBS::CardTableModRefBS()

#### インスタンスの格納場所(where its instances are stored)
なぜか永続的に存在しているインスタンスも存在する.

* 各 AdjoiningVirtualSpaces オブジェクトの _reserved_space フィールド




### 詳細(Details)
See: [here](../doxygen/classReservedSpace.html) for details

---
## <a name="nomXLvzzes" id="nomXLvzzes">ReservedHeapSpace</a>

### 概要(Summary)
特殊な ReservedSpace クラス.
このクラスは Java ヒープ用のメモリ領域の予約に特化している.

(といってもあまり違いはなく UseCompressedOops 機能のための処理が入っている程度.
 0 スタートの領域の予約を試みたり, 駄目だった場合は先頭ページにプロテクションを張ったりする) (See: [here](no289165bb.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/virtualspace.hpp))
    // Class encapsulating behavior specific of memory space reserved for Java heap
    class ReservedHeapSpace : public ReservedSpace {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ParallelScavengeHeap::initialize()
* GenCollectedHeap::allocate()

### 内部構造(Internal structure)
処理としては, スーパークラスである ReservedSpace とほとんど同じ.

#### 参考(for your information): ReservedHeapSpace::ReservedHeapSpace(const size_t prefix_size, const size_t prefix_align, const size_t suffix_size, const size_t suffix_align, char* requested_address)
See: [here](no344Nbk.html) for details
#### 参考(for your information): ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t forced_base_alignment, bool large, char* requested_address)
See: [here](no344Zkj.html) for details



### 詳細(Details)
See: [here](../doxygen/classReservedHeapSpace.html) for details

---
## <a name="no41wp5tEg" id="no41wp5tEg">ReservedCodeSpace</a>

### 概要(Summary)
CodeHeap クラス内で使用される補助クラス.

特殊な ReservedSpace クラス. このクラスは CodeBlob 用のメモリ領域の予約に特化している
(といってもあまり違いはなく, 実行属性が true になっている程度).


```cpp
    ((cite: hotspot/src/share/vm/runtime/virtualspace.hpp))
    // Class encapsulating behavior specific memory space for Code
    class ReservedCodeSpace : public ReservedSpace {
```

### 使われ方(Usage)
CodeHeap::reserve() 内で(のみ)使用されている.

### 内部構造(Internal structure)
処理としては, スーパークラスである ReservedSpace とほとんど同じ.

#### 参考(for your information): ReservedCodeSpace::ReservedCodeSpace()
See: [here](no1904UjT.html) for details



### 詳細(Details)
See: [here](../doxygen/classReservedCodeSpace.html) for details

---
## <a name="noYBaoWzok" id="noYBaoWzok">VirtualSpace</a>

### 概要(Summary)
仮想メモリ空間上の指定した連続領域をコミット(commit)するためのユーティリティ・クラス(ValueObjクラス).

(なお, コミット対象のメモリ空間の予約は ReservedSpace で行う)


```cpp
    ((cite: hotspot/src/share/vm/runtime/virtualspace.hpp))
    // VirtualSpace is data structure for committing a previously reserved address range in smaller chunks.
    
    class VirtualSpace VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
0. あらかじめ, コミットしたい領域を示す ReservedSpace オブジェクトを作っておく.

1. VirtualSpace オブジェクトを生成する

   (現状では, 局所変数としての生成はなく, 基本的に何らかのクラスのフィールドとして(のみ)生成されている)

2. 作っておいた ReservedSpace オブジェクトを引数として VirtualSpace::initialize() を呼び出す.

   (VirtualSpace オブジェクトが初期化されるとともに, 対応するアドレス範囲がコミットされる)

3. VirtualSpace クラスの各種のメソッドで, コミットしたアドレス領域に関する情報が取得できる.

   (また VirtualSpace::expand_by() や VirtualSpace::shrink_by() で後から領域を伸ばす(縮める), といったこともできる)

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 CMSBitMap オブジェクトの _virtual_space フィールド
* 各 CMSMarkStack オブジェクトの _virtual_space フィールド
* 各 CMBitMapRO オブジェクトの _virtual_space フィールド
* 各 G1BlockOffsetSharedArray オブジェクトの _vs フィールド
* 各 G1CollectedHeap オブジェクトの _g1_storage フィールド
* 各 BlockOffsetSharedArray オブジェクトの _vs フィールド
* 各 CompactingPermGenGen オブジェクトの _ro_vs フィールド
* 各 CompactingPermGenGen オブジェクトの _rw_vs フィールド
* 各 CompactingPermGenGen オブジェクトの _md_vs フィールド
* 各 CompactingPermGenGen オブジェクトの _mc_vs フィールド
* 各 Generation オブジェクトの _virtual_space フィールド
* 各 CodeHeap オブジェクトの _memory フィールド
* 各 CodeHeap オブジェクトの _segmap フィールド

#### 生成箇所(where its instances are created)
(上記のフィールドは全て, ポインタ型ではなく実体なので,
 格納しているオブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classVirtualSpace.html) for details

---
