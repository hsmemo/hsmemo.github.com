---
layout: default
title: G1CollectedHeap 用の MemoryPool クラス (G1MemoryPoolSuper, G1EdenPool, G1SurvivorPool, G1OldGenPool)
---
[Top](../index.html)

#### G1CollectedHeap 用の MemoryPool クラス (G1MemoryPoolSuper, G1EdenPool, G1SurvivorPool, G1OldGenPool)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, G1GC 使用時の java.lang.management.MemoryPoolMXBean クラスの実装を担当するクラス.
(See: [here](no2114twV.html) for details)

(なお, G1 では eden/survivor/old といった区別は無いが,
 「eden や survivor に相当する領域がどのくらいの大きさか」という情報は G1 の性能や挙動に影響を与えるので結構重要.)

### 概要(Summary)
G1EdenPool, G1SurvivorPool, G1OldGenPool という 3つのクラスがあり,
それぞれが G1 の HeapRegion 上での eden, survivor, old を仮想的に表現している.

(内部的には, HeapRegion の非連続な集まりからなるメモリプールとして構成されている)

なお, g1MonitoringSupport.hpp 内部のコメントも読むとさらに理解が進む, とのこと.

```cpp
    ((cite: hotspot/src/share/vm/services/g1MemoryPool.hpp))
    // This file contains the three classes that represent the memory
    // pools of the G1 spaces: G1EdenPool, G1SurvivorPool, and
    // G1OldGenPool. In G1, unlike our other GCs, we do not have a
    // physical space for each of those spaces. Instead, we allocate
    // regions for all three spaces out of a single pool of regions (that
    // pool basically covers the entire heap). As a result, the eden,
    // survivor, and old gen are considered logical spaces in G1, as each
    // is a set of non-contiguous regions. This is also reflected in the
    // way we map them to memory pools here. The easiest way to have done
    // this would have been to map the entire G1 heap to a single memory
    // pool. However, it's helpful to show how large the eden and survivor
    // get, as this does affect the performance and behavior of G1. Which
    // is why we introduce the three memory pools implemented here.
    //
    // See comments in g1MonitoringSupport.hpp for additional details
    // on this model.
    //
```



### クラス一覧(class list)

  * [G1MemoryPoolSuper](#noFsI8VueE)
  * [G1EdenPool](#noA1CsDLVV)
  * [G1SurvivorPool](#noCDjsC8Lw)
  * [G1OldGenPool](#noItxnb8hX)


---
## <a name="noFsI8VueE" id="noFsI8VueE">G1MemoryPoolSuper</a>

### 概要(Summary)
G1GC 使用時の全ての MemoryPool クラスの基底クラス. (See: G1EdenPool, G1SurvivorPool, G1OldGenPool)

これらサブクラスの間で共通の処理(used bytes や committed bytes の計算, 等)がこのクラスにまとめている, とのこと.

```cpp
    ((cite: hotspot/src/share/vm/services/g1MemoryPool.hpp))
    // This class is shared by the three G1 memory pool classes
    // (G1EdenPool, G1SurvivorPool, G1OldGenPool). Given that the way we
    // calculate used / committed bytes for these three pools is related
    // (see comment above), we put the calculations in this class so that
    // we can easily share them among the subclasses.
    class G1MemoryPoolSuper : public CollectedMemoryPool {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classG1MemoryPoolSuper.html) for details

---
## <a name="noA1CsDLVV" id="noA1CsDLVV">G1EdenPool</a>

### 概要(Summary)
G1MemoryPoolSuper クラスの具象サブクラスの1つ.

G1GC 使用時の Eden を表す MemoryPool クラス.


```cpp
    ((cite: hotspot/src/share/vm/services/g1MemoryPool.hpp))
    // Memory pool that represents the G1 eden.
    class G1EdenPool : public G1MemoryPoolSuper {
```




### 詳細(Details)
See: [here](../doxygen/classG1EdenPool.html) for details

---
## <a name="noCDjsC8Lw" id="noCDjsC8Lw">G1SurvivorPool</a>

### 概要(Summary)
G1MemoryPoolSuper クラスの具象サブクラスの1つ.

G1GC 使用時の Survivor を表す MemoryPool クラス.


```cpp
    ((cite: hotspot/src/share/vm/services/g1MemoryPool.hpp))
    // Memory pool that represents the G1 survivor.
    class G1SurvivorPool : public G1MemoryPoolSuper {
```




### 詳細(Details)
See: [here](../doxygen/classG1SurvivorPool.html) for details

---
## <a name="noItxnb8hX" id="noItxnb8hX">G1OldGenPool</a>

### 概要(Summary)
G1MemoryPoolSuper クラスの具象サブクラスの1つ.

G1GC 使用時の Old を表す MemoryPool クラス.


```cpp
    ((cite: hotspot/src/share/vm/services/g1MemoryPool.hpp))
    // Memory pool that represents the G1 old gen.
    class G1OldGenPool : public G1MemoryPoolSuper {
```




### 詳細(Details)
See: [here](../doxygen/classG1OldGenPool.html) for details

---
