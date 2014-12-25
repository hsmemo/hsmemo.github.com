---
layout: default
title: ImmutableSpace クラス 
---
[Top](../index.html)

#### ImmutableSpace クラス 



---
## <a name="noLQoRwUFx" id="noLQoRwUFx">ImmutableSpace</a>

### 概要(Summary)
Java ヒープ用のメモリ領域を管理するクラス (の基底クラス) (See: [here](no3718kvd.html) for details).

Java ヒープ用のメモリ領域を管理するクラスは使用する GC アルゴリズムによって異なるが, 
このクラスは GC アルゴリズムが ParallelScavenge の場合に使用される
(ParallelScavenge 用の Space クラス, といった感じ. 実際 Space クラスによく似ている) (See: Space).

bottom() が領域の下端, end() が上端を示す
(これは Space クラスと同様. `bottom() <= end()` という不等式が常に成り立つという点も同じ).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/immutableSpace.hpp))
    // An ImmutableSpace is a viewport into a contiguous range
    // (or subrange) of previously allocated objects.
    
    // Invariant: bottom() and end() are on page_size boundaries and
    // bottom() <= end()
    
    class ImmutableSpace: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classImmutableSpace.html) for details

---
