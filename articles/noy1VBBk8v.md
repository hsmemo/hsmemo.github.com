---
layout: default
title: G1GC で使用する VM_Operation クラス (VM_G1OperationWithAllocRequest, VM_G1CollectFull, VM_G1CollectForAllocation, VM_G1IncCollectionPause, VM_CGC_Operation)
---
[Top](../index.html)

#### G1GC で使用する VM_Operation クラス (VM_G1OperationWithAllocRequest, VM_G1CollectFull, VM_G1CollectForAllocation, VM_G1IncCollectionPause, VM_CGC_Operation)

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, 実際の GC 処理を行うクラス (See: [here](no2480EWm.html) for details).

なお, これらのクラスは以下のような継承関係を持つ
(と, コメントには書いてあるが VM_CGC_Operation は VM_Operation のサブクラスでは??)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    // VM_operations for the G1 collector.
    // VM_GC_Operation:
    //   - VM_CGC_Operation
    //   - VM_G1CollectFull
    //   - VM_G1OperationWithAllocRequest
    //     - VM_G1CollectForAllocation
    //     - VM_G1IncCollectionPause
```



### クラス一覧(class list)

  * [VM_G1OperationWithAllocRequest](#no4hKzSXvm)
  * [VM_G1CollectFull](#noiW9pgFSY)
  * [VM_G1CollectForAllocation](#noxZg3YkTZ)
  * [VM_G1IncCollectionPause](#noT14Bi7FC)
  * [VM_CGC_Operation](#noJnAv1HJA)


---
## <a name="no4hKzSXvm" id="no4hKzSXvm">VM_G1OperationWithAllocRequest</a>

### 概要(Summary)
VM_GC_Operation クラスのサブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, G1CollectedHeap 用.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    class VM_G1OperationWithAllocRequest: public VM_GC_Operation {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classVM__G1OperationWithAllocRequest.html) for details

---
## <a name="noiW9pgFSY" id="noiW9pgFSY">VM_G1CollectFull</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, G1CollectedHeap 用.
(より正確には, G1CollectedHeap において java.lang.System.gc() 等が呼び出された場合用.
GC アルゴリズムとしては G1 MarkSweep がここから呼び出される).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    class VM_G1CollectFull: public VM_GC_Operation {
```

### 使われ方(Usage)
G1CollectedHeap::collect() 内で(のみ)使用されている (See: [here](no28916_jv.html) and [here](no2935eVI.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__G1CollectFull.html) for details

---
## <a name="noxZg3YkTZ" id="noxZg3YkTZ">VM_G1CollectForAllocation</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, G1CollectedHeap 用.
(より正確には, G1CollectedHeap において New/Old 領域からの確保に失敗し, 
さらに VM_G1IncCollectionPause による Minor GC 処理も失敗した場合用.
GC アルゴリズムとしては G1 MarkSweep がここから呼び出される).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    class VM_G1CollectForAllocation: public VM_G1OperationWithAllocRequest {
```

### 使われ方(Usage)
G1CollectedHeap::mem_allocate() 内で(のみ)使用されている (See: [here](no28916fAb.html) and [here](no2935ATn.html) for details).




### 詳細(Details)
See: [here](../doxygen/classVM__G1CollectForAllocation.html) for details

---
## <a name="noT14Bi7FC" id="noT14Bi7FC">VM_G1IncCollectionPause</a>

### 概要(Summary)
VM_GC_Operation クラスの具象サブクラスの1つ (See: [here](no2480EWm.html) for details).

このクラスは, G1CollectedHeap 用.
(より正確には, G1CollectedHeap において New/Old 領域からの確保に失敗した場合用.
ただし, オプションの値によっては System.gc() 等による明示的な GC 処理の場合にも使用される. 
GC アルゴリズムとしては Evacuation Pause がここから呼び出される)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    class VM_G1IncCollectionPause: public VM_G1OperationWithAllocRequest {
```

### 使われ方(Usage)
G1CollectedHeap::do_collection_pause() 内, 及び G1CollectedHeap::collect() 内で(のみ)使用されている
(See: [here](no28916fAb.html), [here](no2935YzN.html), [here](no28916_jv.html) and [here](no2935eVI.html) for details)




### 詳細(Details)
See: [here](../doxygen/classVM__G1IncCollectionPause.html) for details

---
## <a name="noJnAv1HJA" id="noJnAv1HJA">VM_CGC_Operation</a>

### 概要(Summary)
G1CollectedHeap 使用時における Concurrent marking 処理用の補助クラス(VM_Operation クラス).
Concurrent marking 処理中で必要となる Stop-the-World 処理 
(Initial Marking Pause, Final Marking Pause, Live Data Counting & Cleanup) を担当する.

(なお, このクラスは VM_GC_Operation クラスではなく VM_Operation クラスのサブクラス.
 理由は書かれていないが, 恐らく VM_CMS_Operation と同じような理由だと思われる.)

(コメントでは, CMS とコードを共有してみては? と書かれていたりする)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/vm_operations_g1.hpp))
    // Concurrent GC stop-the-world operations such as initial and final mark;
    // consider sharing these with CMS's counterparts.
    class VM_CGC_Operation: public VM_Operation {
```

### 使われ方(Usage)
ConcurrentMarkThread::run() 内で(のみ)使用されている (See: [here](no2935d4w.html) for details).

なお, このクラスは単独では使用できず, 何らかの VoidClosure と併用される
(こいつ自体は実際の処理は行わず, コンストラクタに渡される VoidClosure の do_void() を起動させるだけ.
 これにより Stop-the-World 処理の種別は3種類あるが VM_Operation クラスはこのクラス 1つだけで済んでいる).

現在は以下の VoidClosure と併せて使用される.

* CMCheckpointRootsInitialClosure
* CMCheckpointRootsFinalClosure
* CMCleanUp




### 詳細(Details)
See: [here](../doxygen/classVM__CGC__Operation.html) for details

---
