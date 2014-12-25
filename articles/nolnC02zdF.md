---
layout: default
title: AdjoiningGenerations クラス 
---
[Top](../index.html)

#### AdjoiningGenerations クラス 



---
## <a name="noWTI7rmMi" id="noWTI7rmMi">AdjoiningGenerations</a>

### 概要(Summary)
ParallelScavengeHeap クラス内で使用される補助クラス.

ParallelScavengeHeap 内で使用する generation オブジェクトを管理するためのクラス (See: PSYoungGen, PSOldGen).
具体的には, これらのオブジェクトの確保処理や, これらのオブジェクト間にまたがった領域長の変更処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.hpp))
    // Contains two generations that both use an AdjoiningVirtualSpaces.
    // The two generations are adjacent in the reserved space for the
    // heap.  Each generation has a virtual space and shrinking and
    // expanding of the generations can still be down with that
    // virtual space as was previously done.  If expanding of reserved
    // size of a generation is required, the adjacent generation
    // must be shrunk.  Adjusting the boundary between the generations
    // is called for in this class.
    
    class AdjoiningGenerations : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap クラスの _gens フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      // Collection of generations that are adjacent in the
      // space reserved for the heap.
      AdjoiningGenerations* _gens;
```

#### 使用箇所(where its instances are used)
以下の箇所で使用されている.

* PSYoungGen オブジェクト及び PSOldGen オブジェクトを確保する処理 (See: [here](no2114uSs.html) for details)

  具体的には ParallelScavengeHeap::initialize() 内で使用される.
  この中では, まず AdjoiningGenerations オブジェクトを生成し,
  そこから AdjoiningGenerations::young_gen() と AdjoiningGenerations::old_gen() で
  PSYoungGen オブジェクトと PSOldGen オブジェクトを取り出している.

* PSYoungGen 及び PSOldGen の領域サイズを動的調整する処理 (See: [here](no28916PbD.html) for details)

  具体的には以下の関数内で使用される. これらの関数では必要に応じて Young-Old 領域の境界線をずらしてサイズ調整を行う.

    * ParallelScavengeHeap::resize_young_gen()
    * ParallelScavengeHeap::resize_old_gen()

#### 参考(for your information): ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details
#### 参考(for your information): ParallelScavengeHeap::resize_young_gen()
See: [here](no7882cqe.html) for details
#### 参考(for your information): ParallelScavengeHeap::resize_old_gen()
See: [here](no7882p0k.html) for details
### 内部構造(Internal structure)
このクラスのコンストラクタ内で
PSYoungGen オブジェクトと PSOldGen オブジェクトの生成処理が行われている (See: [here](no2114uSs.html) for details).

#### 参考(for your information): AdjoiningGenerations::AdjoiningGenerations()
See: [here](no344AYS.html) for details



### 詳細(Details)
See: [here](../doxygen/classAdjoiningGenerations.html) for details

---
