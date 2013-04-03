---
layout: default
title: PermGen クラス 
---
[Top](../index.html)

#### PermGen クラス 



---
## <a name="noXgnEqT45" id="noXgnEqT45">PermGen</a>

### 概要(Summary)
Java ヒープ中の「Perm 世代領域 (Perm Generation)」 を管理するためのクラス.

(なお Perm Generation とは, 主にクラスデータを格納するためのメモリ領域)

Perm Generation を管理するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavenge ではない場合に使用されるクラス (の基底クラス)
(See: PSPermGen) (See: [here](no3718kvd.html) for details).


```
    ((cite: hotspot/src/share/vm/memory/permGen.hpp))
    // PermGen models the part of the heap used to allocate class meta-data.
```


```
    ((cite: hotspot/src/share/vm/memory/permGen.hpp))
    class PermGen : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```
    ((cite: hotspot/src/share/vm/memory/permGen.hpp))
      virtual HeapWord* mem_allocate(size_t size) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classPermGen.html) for details

---
