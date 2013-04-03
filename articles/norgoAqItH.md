---
layout: default
title: MemoryPool クラス関連のクラス (MemoryPool, CollectedMemoryPool, ContiguousSpacePool, SurvivorContiguousSpacePool, CompactibleFreeListSpacePool, GenerationPool, CodeHeapPool)
---
[Top](../index.html)

#### MemoryPool クラス関連のクラス (MemoryPool, CollectedMemoryPool, ContiguousSpacePool, SurvivorContiguousSpacePool, CompactibleFreeListSpacePool, GenerationPool, CodeHeapPool)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.MemoryPoolMXBean クラスの実装を担当するクラス.
(See: [here](no2114twV.html) for details)
(See: [here](no211477i.html) for details)


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    // A memory pool represents the memory area that the VM manages.
    // The Java virtual machine has at least one memory pool
    // and it may create or remove memory pools during execution.
    // A memory pool can belong to the heap or the non-heap memory.
    // A Java virtual machine may also have memory pools belonging to
    // both heap and non-heap memory.
```



### クラス一覧(class list)

  * [MemoryPool](#nog84egjPN)
  * [CollectedMemoryPool](#nojYZXh46W)
  * [ContiguousSpacePool](#noYyQueS-F)
  * [SurvivorContiguousSpacePool](#noEhFCM_vS)
  * [CompactibleFreeListSpacePool](#nogMwZ2EAq)
  * [GenerationPool](#noRx1PaGbZ)
  * [CodeHeapPool](#nodbfeamFi)


---
## <a name="nog84egjPN" id="nog84egjPN">MemoryPool</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: java.lang.management.MemoryPoolMXBean)
(See: [here](no2114twV.html) and [here](no211477i.html) for details).

全ての MemoryPool クラスの基底クラス.


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class MemoryPool : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
      virtual MemoryUsage get_memory_usage() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classMemoryPool.html) for details

---
## <a name="nojYZXh46W" id="nojYZXh46W">CollectedMemoryPool</a>

### 概要(Summary)
MemoryPool クラスのサブクラスの1つ (See: [here](no2114twV.html) and [here](no211477i.html) for details).

このクラスは, CollectedHeap の領域用.


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class CollectedMemoryPool : public MemoryPool {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classCollectedMemoryPool.html) for details

---
## <a name="noYyQueS-F" id="noYyQueS-F">ContiguousSpacePool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 使用時の Eden 領域(等)用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class ContiguousSpacePool : public CollectedMemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classContiguousSpacePool.html) for details

---
## <a name="noEhFCM_vS" id="noEhFCM_vS">SurvivorContiguousSpacePool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 使用時の Survivor 領域用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class SurvivorContiguousSpacePool : public CollectedMemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classSurvivorContiguousSpacePool.html) for details

---
## <a name="nogMwZ2EAq" id="nogMwZ2EAq">CompactibleFreeListSpacePool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, CMS 使用時の Old 領域および Perm 領域用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class CompactibleFreeListSpacePool : public CollectedMemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classCompactibleFreeListSpacePool.html) for details

---
## <a name="noRx1PaGbZ" id="noRx1PaGbZ">GenerationPool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, MarkSweepCompact (= Serial Old) 使用時の Old 領域用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class GenerationPool : public CollectedMemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classGenerationPool.html) for details

---
## <a name="nodbfeamFi" id="nodbfeamFi">CodeHeapPool</a>

### 概要(Summary)
MemoryPool クラスの具象サブクラスの1つ.

このクラスは, CodeHeap が使用するメモリ領域用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class CodeHeapPool: public MemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classCodeHeapPool.html) for details

---
