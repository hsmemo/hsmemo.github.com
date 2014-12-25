---
layout: default
title: GenRemSet クラス 
---
[Top](../index.html)

#### GenRemSet クラス 



---
## <a name="nok9XVTzqQ" id="nok9XVTzqQ">GenRemSet</a>

### 概要(Summary)
Garbage Collection 処理用の補助クラス.
より具体的に言うと, Remembered Set 機能を提供するクラス (の基底クラス) (See: [here](no3718kvd.html) for details).

Remembered Set 機能を提供するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavengeHeap 以外の場合に使用される (See: CardTableExtension).

(<= ただし, G1CollectedHeap の場合も主に別のクラスが Remembered Set 機能を担当するので, 
実質的には GenCollectedHeap でしか使われていない模様 #TODO) (See: HeapRegionRemSet, G1RemSet)


```cpp
    ((cite: hotspot/src/share/vm/memory/genRemSet.hpp))
    // A GenRemSet provides ways of iterating over pointers accross generations.
    // (This is especially useful for older-to-younger.)
```


```cpp
    ((cite: hotspot/src/share/vm/memory/genRemSet.hpp))
    class GenRemSet: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/genRemSet.hpp))
      virtual Name rs_kind() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classGenRemSet.html) for details

---
