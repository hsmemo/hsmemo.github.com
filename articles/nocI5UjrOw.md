---
layout: default
title: ParallelScavengeHeap クラス関連のクラス (ParallelScavengeHeap, ParallelScavengeHeap::ParStrongRootsScope)
---
[Top](../index.html)

#### ParallelScavengeHeap クラス関連のクラス (ParallelScavengeHeap, ParallelScavengeHeap::ParStrongRootsScope)

これらは, Java ヒープ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが ParallelScavenge の場合に使用される.
(See: GenCollectedHeap, G1CollectedHeap)



### クラス一覧(class list)

  * [ParallelScavengeHeap](#noWu6EIS0n)
  * [ParallelScavengeHeap::ParStrongRootsScope](#noOKnCVFtm)


---
## <a name="noWu6EIS0n" id="noWu6EIS0n">ParallelScavengeHeap</a>

### 概要(Summary)
Java ヒープ領域の管理を担当するクラス(CollectedHeapクラス)の1つ (See: [here](no3718kvd.html) for details).

このクラスは, GC アルゴリズムが ParallelScavenge の場合用.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
```




### 詳細(Details)
See: [here](../doxygen/classParallelScavengeHeap.html) for details

---
## <a name="noOKnCVFtm" id="noOKnCVFtm">ParallelScavengeHeap::ParStrongRootsScope</a>

### 概要(Summary)
ParallelScavenge 用の MarkingCodeBlobClosure::MarkScope クラス (See: MarkingCodeBlobClosure::MarkScope).

ただし, 内部の処理的には MarkingCodeBlobClosure::MarkScope と全く同じ
(MarkingCodeBlobClosure::MarkScope と違ってこちらは abstract class ではないが).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
      // Call these in sequential code around the processing of strong roots.
      class ParStrongRootsScope : public MarkingCodeBlobClosure::MarkScope {
```

### 内部構造(Internal structure)
(中身は MarkingCodeBlobClosure::MarkScope() から変更無し)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp))
    ParallelScavengeHeap::ParStrongRootsScope::ParStrongRootsScope() {
      // nothing particular
    }
    
    ParallelScavengeHeap::ParStrongRootsScope::~ParStrongRootsScope() {
      // nothing particular
    }
```




### 詳細(Details)
See: [here](../doxygen/classParallelScavengeHeap_1_1ParStrongRootsScope.html) for details

---
