---
layout: default
title: ModRefBarrierSet クラス 
---
[Top](../index.html)

#### ModRefBarrierSet クラス 



---
## <a name="noGU1LKka3" id="noGU1LKka3">ModRefBarrierSet</a>

### 概要(Summary)
Garbage Collection 処理用の補助クラス (の基底クラス). 
より具体的に言うと, BarrierSet の一種 (See: [here](no3718kvd.html) for details).

CollectedHeap 内で変更されたポインタ(参照)フィールドを検出する機能と, 
その変更箇所に対して iterate する機能を提供する.

(なお, 内部的には card table を使用する)

```
    ((cite: hotspot/src/share/vm/memory/modRefBarrierSet.hpp))
    // This kind of "BarrierSet" allows a "CollectedHeap" to detect and
    // enumerate ref fields that have been modified (since the last
    // enumeration), using a card table.
```


```
    ((cite: hotspot/src/share/vm/memory/modRefBarrierSet.hpp))
    class ModRefBarrierSet: public BarrierSet {
```


なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/modRefBarrierSet.hpp))
      virtual bool write_ref_needs_barrier(void* field, oop new_val) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classModRefBarrierSet.html) for details

---
