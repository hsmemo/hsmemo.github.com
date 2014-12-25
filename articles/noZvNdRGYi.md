---
layout: default
title: BlockOffsetTable クラス関連のクラス (BlockOffsetTable, BlockOffsetSharedArray, BlockOffsetArray, BlockOffsetArrayNonContigSpace, BlockOffsetArrayContigSpace)
---
[Top](../index.html)

#### BlockOffsetTable クラス関連のクラス (BlockOffsetTable, BlockOffsetSharedArray, BlockOffsetArray, BlockOffsetArrayNonContigSpace, BlockOffsetArrayContigSpace)

これらは, Java ヒープのメモリ管理用のクラス.
より具体的に言うと, card 方式の BarrierSet を補佐するためのクラス.

### 概要(Summary)
card 方式の BarrierSet では Java ヒープ (CollectedHeap) 中の変更を card 単位で記録している (See: CardTableModRefBS).

ところが, Java ヒープ中のオブジェクトは card 単位でアラインメントされてはいないので card の境界をまたぐようなオブジェクトも存在しうる.
そのため card 内をいきなり見ても「どこがオブジェクトの切れ目か」を示す手がかりが全くなく使いにくい.

そこで, BlockOffsetTable というオブジェクト (あるいはそのサブクラスのオブジェクト) に
「各 card 内における先頭オブジェクトの位置(オフセット)」 を記録している.
これを使うことでオブジェクトの頭出しができるようになり, card 情報が役に立つようになる.

より具体的に言うと, この「頭出し」機能は CollectedHeap::block_start() メソッドとして実装されている.
そのため BlockOffsetTable クラス(やそのサブクラス) はこのメソッド用の補助クラスともいえる.

(BlockOffsetTable 情報を有効活用しているクラスとしては CardGeneration や OffsetTableContigSpace 等)

(<= とは言え, この仕組み自体は確かに CollectedHeap クラスの全てのサブクラスで共通だが, 
 ParallelScavengeHeap/G1CollectedHeap では BlockOffsetTable とは別のクラスが使われているので, 
 BlockOffsetTable クラス自体は GenCollectedHeap 専用のクラスになっている...
 (See: ObjectStartArray, G1BlockOffsetTable))

なお, これらのクラスは以下のような継承関係を持つ.

  * BlockOffsetTable                         (abstract class)
      * BlockOffsetArray                     (abstract class)
          * BlockOffsetArrayNonContigSpace   (CMS 用)
          * BlockOffsetArrayContigSpace      (CMS 以外用)


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    // The CollectedHeap type requires subtypes to implement a method
    // "block_start".  For some subtypes, notably generational
    // systems using card-table-based write barriers, the efficiency of this
    // operation may be important.  Implementations of the "BlockOffsetArray"
    // class may be useful in providing such efficient implementations.
```


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    // BlockOffsetTable (abstract)
    //   - BlockOffsetArray (abstract)
    //     - BlockOffsetArrayNonContigSpace
    //     - BlockOffsetArrayContigSpace
```



### クラス一覧(class list)

  * [BlockOffsetTable](#noNQWTbdP8)
  * [BlockOffsetSharedArray](#no4fUjSM8d)
  * [BlockOffsetArray](#noQs5ZGy-7)
  * [BlockOffsetArrayNonContigSpace](#nocDj8qb-u)
  * [BlockOffsetArrayContigSpace](#nouJIaJFX9)


---
## <a name="noNQWTbdP8" id="noNQWTbdP8">BlockOffsetTable</a>

### 概要(Summary)
card 方式の BarrierSet を補佐するためのクラス (の基底クラス) (See: CardTableModRefBS).

「各 card 内における先頭オブジェクトの位置(オフセット)」を格納するためのクラス. 
card 内のオブジェクトの頭出しを高速化するために使用される.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    //////////////////////////////////////////////////////////////////////////
    // The BlockOffsetTable "interface"
    //////////////////////////////////////////////////////////////////////////
    class BlockOffsetTable VALUE_OBJ_CLASS_SPEC {
```


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
      virtual HeapWord* block_start_unsafe(const void* addr) const = 0;
```



### 詳細(Details)
See: [here](../doxygen/classBlockOffsetTable.html) for details

---
## <a name="no4fUjSM8d" id="no4fUjSM8d">BlockOffsetSharedArray</a>

### 概要(Summary)
BlockOffsetArray クラス (のサブクラス) 内で使用される補助クラス.

BlockOffsetArray では, 対象の領域を 2^N の大きさの部分領域に分割して管理している.
そして内部的にはその情報を配列で管理している. この配列が BlockOffsetSharedArray オブジェクト.

BlockOffsetArray オブジェクト自体は Space オブジェクトと １対１対応だが, 
内部で使う配列は複数の BlockOffsetArray オブジェクトで共有した方がいい場合もある (G1GC みたいな場合など).
そのため内部で使う配列だけを別クラスとしている, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    //////////////////////////////////////////////////////////////////////////
    // BlockOffsetSharedArray
    //////////////////////////////////////////////////////////////////////////
    class BlockOffsetSharedArray: public CHeapObj {
```


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    //////////////////////////////////////////////////////////////////////////
    // One implementation of "BlockOffsetTable," the BlockOffsetArray,
    // divides the covered region into "N"-word subregions (where
    // "N" = 2^"LogN".  An array with an entry for each such subregion
    // indicates how far back one must go to find the start of the
    // chunk that includes the first word of the subregion.
    //
    // Each BlockOffsetArray is owned by a Space.  However, the actual array
    // may be shared by several BlockOffsetArrays; this is useful
    // when a single resizable area (such as a generation) is divided up into
    // several spaces in which contiguous allocation takes place.  (Consider,
    // for example, the garbage-first generation.)
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
各 CardGeneration 内で, 対応する BlockOffsetSharedArray オブジェクトが生成され, 
インスタンスフィールドに格納されている.

(そして, BlockOffsetArray オブジェクトを作る際のコンストラクタ引数として用いられる)


```cpp
    ((cite: hotspot/src/share/vm/memory/generation.cpp))
    CardGeneration::CardGeneration(ReservedSpace rs, size_t initial_byte_size,
                                   int level,
                                   GenRemSet* remset) :
    ...
      _bts = new BlockOffsetSharedArray(reserved_mr,
                                        heap_word_size(initial_byte_size));
```



### 詳細(Details)
See: [here](../doxygen/classBlockOffsetSharedArray.html) for details

---
## <a name="noQs5ZGy-7" id="noQs5ZGy-7">BlockOffsetArray</a>

### 概要(Summary)
全ての BlockOffsetArray クラスの基底クラス.

スーパークラスである BlockOffsetTable クラスとの違いは, 
内部的に BlockOffsetSharedArray を活用するという点

(<= そのため, 対象の領域を 2^N の大きさの部分領域に分割して管理している, という違いもある).


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    //////////////////////////////////////////////////////////////////////////
    // The BlockOffsetArray whose subtypes use the BlockOffsetSharedArray.
    //////////////////////////////////////////////////////////////////////////
    class BlockOffsetArray: public BlockOffsetTable {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
      virtual size_t last_active_index() const = 0;
```



### 詳細(Details)
See: [here](../doxygen/classBlockOffsetArray.html) for details

---
## <a name="nocDj8qb-u" id="nocDj8qb-u">BlockOffsetArrayNonContigSpace</a>

### 概要(Summary)
BlockOffsetArray クラスの具象サブクラスの1つ.

このクラスは不連続な領域用 (もっと具体的に言うと CMS 用).


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    ////////////////////////////////////////////////////////////////////////////
    // A subtype of BlockOffsetArray that takes advantage of the fact
    // that its underlying space is a NonContiguousSpace, so that some
    // specialized interfaces can be made available for spaces that
    // manipulate the table.
    ////////////////////////////////////////////////////////////////////////////
    class BlockOffsetArrayNonContigSpace: public BlockOffsetArray {
```

### 使われ方(Usage)
現状では CMS の CompactibleFreeListSpace の中でしか使われていない？ #TODO



### 詳細(Details)
See: [here](../doxygen/classBlockOffsetArrayNonContigSpace.html) for details

---
## <a name="nouJIaJFX9" id="nouJIaJFX9">BlockOffsetArrayContigSpace</a>

### 概要(Summary)
BlockOffsetArray クラスの具象サブクラスの1つ.

このクラスは連続な領域用 (もっと具体的に言うと CMS 以外用).
連続しているという仮定により, BlockOffsetArrayNonContigSpace よりも効率的にメモリ管理ができる.


```cpp
    ((cite: hotspot/src/share/vm/memory/blockOffsetTable.hpp))
    ////////////////////////////////////////////////////////////////////////////
    // A subtype of BlockOffsetArray that takes advantage of the fact
    // that its underlying space is a ContiguousSpace, so that its "active"
    // region can be more efficiently tracked (than for a non-contiguous space).
    ////////////////////////////////////////////////////////////////////////////
    class BlockOffsetArrayContigSpace: public BlockOffsetArray {
```

### 使われ方(Usage)
以下のクラス内で(のみ)使われている模様.

  * OffsetTableContigSpace
  * ParGCAllocBufferWithBOT (<= ParNew 用)




### 詳細(Details)
See: [here](../doxygen/classBlockOffsetArrayContigSpace.html) for details

---
