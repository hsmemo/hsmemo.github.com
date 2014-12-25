---
layout: default
title: MemoryManager クラス関連のクラス (MemoryManager, CodeCacheMemoryManager, GCStatInfo, GCMemoryManager, CopyMemoryManager, MSCMemoryManager, ParNewMemoryManager, CMSMemoryManager, PSScavengeMemoryManager, PSMarkSweepMemoryManager, G1YoungGenMemoryManager, G1OldGenMemoryManager)
---
[Top](../index.html)

#### MemoryManager クラス関連のクラス (MemoryManager, CodeCacheMemoryManager, GCStatInfo, GCMemoryManager, CopyMemoryManager, MSCMemoryManager, ParNewMemoryManager, CMSMemoryManager, PSScavengeMemoryManager, PSMarkSweepMemoryManager, G1YoungGenMemoryManager, G1OldGenMemoryManager)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, java.lang.management.MemoryManagerMXBean クラスの実装を担当するクラス.
(See: [here](no2114twV.html) for details)
(See: [here](no211477i.html) for details)

### 備考(Notes)
GCMemoryManager クラスには各 GC アルゴリズム用のサブクラスが存在する. 

これらは GC に特化した情報を含めるために作られた, とのこと.
(ただし, 現状では name() と kind() の返値が違う程度の差しかない...
 コメントでは, TODO 事項として各 GC に特化した情報を入れるように, と書かれているが...)

```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    // These subclasses of GCMemoryManager are defined to include
    // GC-specific information.
    // TODO: Add GC-specific information
```



### クラス一覧(class list)

  * [MemoryManager](#nog8DPCMct)
  * [CodeCacheMemoryManager](#noLehuZ72D)
  * [GCStatInfo](#noA56c1oMF)
  * [GCMemoryManager](#noO9BGzWHV)
  * [CopyMemoryManager](#nonIn7PYSC)
  * [MSCMemoryManager](#noJ5BsDDGm)
  * [ParNewMemoryManager](#nouDmRloBc)
  * [CMSMemoryManager](#nomj8GJt0h)
  * [PSScavengeMemoryManager](#now-5xCoE1)
  * [PSMarkSweepMemoryManager](#no_3kux_Oa)
  * [G1YoungGenMemoryManager](#nop-QK2WBM)
  * [G1OldGenMemoryManager](#noEop0JsX6)


---
## <a name="nog8DPCMct" id="nog8DPCMct">MemoryManager</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: java.lang.management.MemoryManagerMXBean)
(See: [here](no2114twV.html) and [here](no211477i.html) for details).

全ての MemoryManager クラスの基底クラス 


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class MemoryManager : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
      virtual const char* name() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classMemoryManager.html) for details

---
## <a name="noLehuZ72D" id="noLehuZ72D">CodeCacheMemoryManager</a>

### 概要(Summary)
MemoryManager クラスの具象サブクラスの1つ.

このクラスは, CodeCache が使用するメモリ領域用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class CodeCacheMemoryManager : public MemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classCodeCacheMemoryManager.html) for details

---
## <a name="noA56c1oMF" id="noA56c1oMF">GCStatInfo</a>

### 概要(Summary)
保守運用機能のためのクラス (JMM 機能用のクラス).

com.sun.management.GcInfo を実現するためのクラス (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class GCStatInfo : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classGCStatInfo.html) for details

---
## <a name="noO9BGzWHV" id="noO9BGzWHV">GCMemoryManager</a>

### 概要(Summary)
MemoryManager クラスのサブクラスの1つ.
GC 処理に関する MemoryManager クラス (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class GCMemoryManager : public MemoryManager {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
      virtual MemoryManager::Name kind() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classGCMemoryManager.html) for details

---
## <a name="nonIn7PYSC" id="nonIn7PYSC">CopyMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 上での Copy GC (= Serial GC) 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class CopyMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classCopyMemoryManager.html) for details

---
## <a name="noJ5BsDDGm" id="noJ5BsDDGm">MSCMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 上での MarkSweepCompact GC (= Serial Old GC) 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class MSCMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classMSCMemoryManager.html) for details

---
## <a name="nouDmRloBc" id="nouDmRloBc">ParNewMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 上での ParNew GC 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class ParNewMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classParNewMemoryManager.html) for details

---
## <a name="nomj8GJt0h" id="nomj8GJt0h">CMSMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, GenCollectedHeap 上での CMS GC 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class CMSMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classCMSMemoryManager.html) for details

---
## <a name="now-5xCoE1" id="now-5xCoE1">PSScavengeMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, ParallelScavengeHeap 上での Minor GC (Scavenge GC) 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class PSScavengeMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classPSScavengeMemoryManager.html) for details

---
## <a name="no_3kux_Oa" id="no_3kux_Oa">PSMarkSweepMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, ParallelScavengeHeap 上での Major GC (MarkSweep GC) 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class PSMarkSweepMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classPSMarkSweepMemoryManager.html) for details

---
## <a name="nop-QK2WBM" id="nop-QK2WBM">G1YoungGenMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, G1CollectedHeap 上での Minor GC (Evacuation Pause) 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class G1YoungGenMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classG1YoungGenMemoryManager.html) for details

---
## <a name="noEop0JsX6" id="noEop0JsX6">G1OldGenMemoryManager</a>

### 概要(Summary)
GCMemoryManager クラスの具象サブクラスの1つ.

このクラスは, G1CollectedHeap 上での Major GC 用 (See: [here](no2114twV.html) and [here](no211477i.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/services/memoryManager.hpp))
    class G1OldGenMemoryManager : public GCMemoryManager {
```




### 詳細(Details)
See: [here](../doxygen/classG1OldGenMemoryManager.html) for details

---
