---
layout: default
title: PSAdaptiveSizePolicy クラス 
---
[Top](../index.html)

#### PSAdaptiveSizePolicy クラス 



---
## <a name="noqp7-ki6u" id="noqp7-ki6u">PSAdaptiveSizePolicy</a>

### 概要(Summary)
ParallelScavenge 用の AdaptiveSizePolicy クラス 
(つまり, Java ヒープの動的なサイズ変更処理時に新しいヒープサイズの計算を行うクラス (See: AdaptiveSizePolicy)).


このクラスでは, Old generation と Young generation の大きさのほかに, 
適切な tenuring threshold の値や survivor space の大きさも計算している.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psAdaptiveSizePolicy.hpp))
    // This class keeps statistical information and computes the
    // optimal free space for both the young and old generation
    // based on current application characteristics (based on gc cost
    // and application footprint).
    //
    // It also computes an optimal tenuring threshold between the young
    // and old generations, so as to equalize the cost of collections
    // of those generations, as well as optimial survivor space sizes
    // for the young generation.
    //
    // While this class is specifically intended for a generational system
    // consisting of a young gen (containing an Eden and two semi-spaces)
    // and a tenured gen, as well as a perm gen for reflective data, it
    // makes NO references to specific generations.
    //
    // 05/02/2003 Update
    // The 1.5 policy makes use of data gathered for the costs of GC on
    // specific generations.  That data does reference specific
    // generation.  Also diagnostics specific to generations have
    // been added.
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psAdaptiveSizePolicy.hpp))
    class PSAdaptiveSizePolicy : public AdaptiveSizePolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap クラスの _size_policy フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      // Sizing policy for entire heap
      static PSAdaptiveSizePolicy* _size_policy;
```




### 詳細(Details)
See: [here](../doxygen/classPSAdaptiveSizePolicy.html) for details

---
