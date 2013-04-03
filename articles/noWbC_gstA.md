---
layout: default
title: ageTable クラス 
---
[Top](../index.html)

#### ageTable クラス 



---
## <a name="noSgyKBAJr" id="noSgyKBAJr">ageTable</a>

### 概要(Summary)
Generational GC における昇格のための閾値(tenuring threshould)を計算するクラス.

昇格閾値を計算するクラスは使用する GC アルゴリズムによって異なるが, 
このクラスは Minor GC アルゴリズムが Serial, ParNew, G1GC の場合に使用される
(ParallelScavenge では
 PSAdaptiveSizePolicy::compute_survivor_space_size_and_threshold() で計算しており
 ageTable は使われていない模様).


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp))
    // Age table for adaptive feedback-mediated tenuring (scavenging)
    //
    // Note: all sizes are in oops
    
    class ageTable VALUE_OBJ_CLASS_SPEC {
```


### 使われ方(Usage)
#### 使用方法の概要(how to use)

1. まず, GC 開始時に ageTable::clear() を呼んでおく(??)


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp))
      // clear table
      void clear();
```

2. GC 中に, ageTable::add() を呼んで live object の情報を記録していく.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp))
      // add entry
      void add(oop p, size_t oop_size) {
```

2. (ParNew では, 複数の ageTable をマージする ageTable::merge() や ageTable::merge_par() も使用されている)


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp))
      // Merge another age table with the current one.  Used
      // for parallel young generation gc.
      void merge(ageTable* subTable);
      void merge_par(ageTable* subTable);
```

3. そして, GC 終了後に ageTable::compute_tenuring_threshold() を呼ぶと, 次の昇格閾値が計算されて返される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/ageTable.hpp))
      // calculate new tenuring threshold based on age information
      int compute_tenuring_threshold(size_t survivor_capacity);
```




### 詳細(Details)
See: [here](../doxygen/classageTable.html) for details

---
