---
layout: default
title: PSPromotionLAB 及びそのサブクラス (PSPromotionLAB, PSYoungPromotionLAB, PSOldPromotionLAB)
---
[Top](../index.html)

#### PSPromotionLAB 及びそのサブクラス (PSPromotionLAB, PSYoungPromotionLAB, PSOldPromotionLAB)

これらは, ParallelScavengeHeap の Minor GC 処理
("Parallel Scavenge" 処理) で使用される補助クラス (See: [here](no289165Un.html) for details).

より具体的に言うと, Minor GC 時 (Copy GC 時) に
各 GCTaskThread がコピー先として使用する To 領域や Old 領域を管理するクラス.


### クラス一覧(class list)

  * [PSPromotionLAB](#nokr8qADQo)
  * [PSYoungPromotionLAB](#noVCuyveUV)
  * [PSOldPromotionLAB](#no74hJozPC)


---
## <a name="nokr8qADQo" id="nokr8qADQo">PSPromotionLAB</a>

### 概要(Summary)
ParallelScavengeHeap の Minor GC 処理で使用される "Local Allocation Buffer" クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.hpp))
    class PSPromotionLAB : public CHeapObj {
```

各 GCTaskThread がコピー先として使用する To 領域と Old 領域を管理するクラス.

コピーするたびにメモリ領域を排他で取得したりすると重いので, 
あらかじめある程度の領域を thread local に確保するためのもの.
主に PSPromotionManager 内で使用される (See: PSPromotionManager).

なお, 中身は MutableSpace にそっくりだが, 
MutableSpace 内の assert 等が Parallel Scavenge 処理での使い方に合わないため別にクラスを作った, 
とのこと.

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.hpp))
    // PSPromotionLAB is a parallel scavenge promotion lab. This class acts very
    // much like a MutableSpace. We couldn't embed a MutableSpace, though, as
    // it has a considerable number of asserts and invariants that are violated.
```




### 詳細(Details)
See: [here](../doxygen/classPSPromotionLAB.html) for details

---
## <a name="noVCuyveUV" id="noVCuyveUV">PSYoungPromotionLAB</a>

### 概要(Summary)
PSPromotionLAB クラスの具象サブクラスの1つ.

このクラスは To 領域用
(つまり各 GCTaskThread がコピー先として使用する To 領域をある程度 thread local で確保しておくためのクラス).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.hpp))
    class PSYoungPromotionLAB : public PSPromotionLAB {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSPromotionManager オブジェクトの _young_lab フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp))
      PSYoungPromotionLAB                 _young_lab;
```




### 詳細(Details)
See: [here](../doxygen/classPSYoungPromotionLAB.html) for details

---
## <a name="no74hJozPC" id="no74hJozPC">PSOldPromotionLAB</a>

### 概要(Summary)
PSPromotionLAB クラスの具象サブクラスの1つ.

このクラスは Old 領域用
(つまり各 GCTaskThread がコピー先として使用する Old 領域をある程度 thread local で確保しておくためのクラス).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionLAB.hpp))
    class PSOldPromotionLAB : public PSPromotionLAB {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSPromotionManager オブジェクトの _old_lab フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPromotionManager.hpp))
      PSOldPromotionLAB                   _old_lab;
```




### 詳細(Details)
See: [here](../doxygen/classPSOldPromotionLAB.html) for details

---
