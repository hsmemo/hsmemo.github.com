---
layout: default
title: ParallelScavengeHeap 用の MemoryPool クラス (PSGenerationPool, EdenMutableSpacePool, SurvivorMutableSpacePool)
---
[Top](../index.html)

#### ParallelScavengeHeap 用の MemoryPool クラス (PSGenerationPool, EdenMutableSpacePool, SurvivorMutableSpacePool)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, ParallelScavengeHeap 使用時の java.lang.management.MemoryPoolMXBean クラスの実装を担当するクラス.
(See: [here](no2114twV.html) for details)


### クラス一覧(class list)

  * [PSGenerationPool](#noK9OwsM92)
  * [EdenMutableSpacePool](#noS7JLnXEL)
  * [SurvivorMutableSpacePool](#noQtoLXF7W)


---
## <a name="noK9OwsM92" id="noK9OwsM92">PSGenerationPool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, ParallelScavenge 使用時の Old/Perm 領域用 (See: [here](no2114twV.html) for details).


```
    ((cite: hotspot/src/share/vm/services/psMemoryPool.hpp))
    class PSGenerationPool : public CollectedMemoryPool {
```



### 詳細(Details)
See: [here](../doxygen/classPSGenerationPool.html) for details

---
## <a name="noS7JLnXEL" id="noS7JLnXEL">EdenMutableSpacePool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, ParallelScavenge 使用時の Eden 領域用 (See: [here](no2114twV.html) for details).


```
    ((cite: hotspot/src/share/vm/services/psMemoryPool.hpp))
    class EdenMutableSpacePool : public CollectedMemoryPool {
```



### 詳細(Details)
See: [here](../doxygen/classEdenMutableSpacePool.html) for details

---
## <a name="noQtoLXF7W" id="noQtoLXF7W">SurvivorMutableSpacePool</a>

### 概要(Summary)
CollectedMemoryPool クラスの具象サブクラスの1つ.

このクラスは, ParallelScavenge 使用時の Survivor 領域用 (See: [here](no2114twV.html) for details).


```
    ((cite: hotspot/src/share/vm/services/psMemoryPool.hpp))
    class SurvivorMutableSpacePool : public CollectedMemoryPool {
```




### 詳細(Details)
See: [here](../doxygen/classSurvivorMutableSpacePool.html) for details

---
