---
layout: default
title: ParallelScavenge 用の VM_GC_Operation のサブクラス (VM_ParallelGCFailedAllocation, VM_ParallelGCFailedPermanentAllocation, VM_ParallelGCSystemGC)
---
[Top](../index.html)

#### ParallelScavenge 用の VM_GC_Operation のサブクラス (VM_ParallelGCFailedAllocation, VM_ParallelGCFailedPermanentAllocation, VM_ParallelGCSystemGC)

これらは, ParallelScavengeHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, 実際の GC 処理を行うクラス (See: [here](no2480EWm.html) for details).


### クラス一覧(class list)

  * [VM_ParallelGCFailedAllocation](#noEZC4J0nj)
  * [VM_ParallelGCFailedPermanentAllocation](#no2ZkpVgbH)
  * [VM_ParallelGCSystemGC](#noLCt0m75s)


---
## <a name="noEZC4J0nj" id="noEZC4J0nj">VM_ParallelGCFailedAllocation</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, ParallelScavengeHeap 用
(より正確には, ParallelScavengeHeap において New/Old 領域からの確保に失敗した場合用.
GC アルゴリズムとしては Parallel Scavenge, Parallel Compaction, PS MarkSweep がここから呼び出される).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.hpp))
    class VM_ParallelGCFailedAllocation: public VM_GC_Operation {
```

### 使われ方(Usage)
ParallelScavengeHeap::mem_allocate() 内で(のみ)使用されている (See: [here](no3718vrX.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__ParallelGCFailedAllocation.html) for details

---
## <a name="no2ZkpVgbH" id="no2ZkpVgbH">VM_ParallelGCFailedPermanentAllocation</a>

VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, ParallelScavengeHeap 用
(より正確には, ParallelScavengeHeap において Perm 領域からの確保に失敗した場合用.
GC アルゴリズムとしては Parallel Compaction, PS MarkSweep がここから呼び出される).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.hpp))
    class VM_ParallelGCFailedPermanentAllocation: public VM_GC_Operation {
```

### 使われ方(Usage)
ParallelScavengeHeap::permanent_mem_allocate() 内で(のみ)使用されている (See: [here](no28916-pc.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__ParallelGCFailedPermanentAllocation.html) for details

---
## <a name="noLCt0m75s" id="noLCt0m75s">VM_ParallelGCSystemGC</a>

VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, ParallelScavengeHeap 用
(より正確には, ParallelScavengeHeap において java.lang.System.gc() 等が呼び出された場合用.
GC アルゴリズムとしては Parallel Compaction, PS MarkSweep がここから呼び出される).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/vmPSOperations.hpp))
    class VM_ParallelGCSystemGC: public VM_GC_Operation {
```

### 使われ方(Usage)
ParallelScavengeHeap::collect() 内で(のみ)使用されている (See: [here](no28916_jv.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__ParallelGCSystemGC.html) for details

---
