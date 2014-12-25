---
layout: default
title: MutableSpace クラス 
---
[Top](../index.html)

#### MutableSpace クラス 



---
## <a name="noHhVOgoOS" id="noHhVOgoOS">MutableSpace</a>

### 概要(Summary)
Java ヒープ用のメモリ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ用のメモリ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
このクラスは GC アルゴリズムが ParallelScavenge の場合に使用される
(ParallelScavenge 用の ContiguousSpace クラス, といった感じ) (See: ContiguousSpace).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableSpace.hpp))
    class MutableSpace: public ImmutableSpace {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavenge 用の Java ヒープを管理するオブジェクト内 (PSYoungGen, PSOldGen, MutableNUMASpace::LGRPSpace) に格納されている.

* PSYoungGen

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.hpp))
    class PSYoungGen : public CHeapObj {
    ...
      // Spaces
      MutableSpace* _eden_space;
      MutableSpace* _from_space;
      MutableSpace* _to_space;
```

* PSOldGen

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp))
    class PSOldGen : public CHeapObj {
    ...
      MutableSpace*            _object_space;      // Where all the objects live
```

* MutableNUMASpace::LGRPSpace

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp))
      class LGRPSpace : public CHeapObj {
    ...
        MutableSpace* _space;
```

#### 生成箇所(where its instances are created)
それぞれ, 以下の箇所で生成されている.

* PSYoungGen::initialize_work()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psYoungGen.cpp))
    void PSYoungGen::initialize_work() {
    ...
      if (UseNUMA) {
        _eden_space = new MutableNUMASpace(virtual_space()->alignment());
      } else {
        _eden_space = new MutableSpace(virtual_space()->alignment());
      }
      _from_space = new MutableSpace(virtual_space()->alignment());
      _to_space   = new MutableSpace(virtual_space()->alignment());
```

* PSOldGen::initialize_work()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.cpp))
    void PSOldGen::initialize_work(const char* perf_data_name, int level) {
    ...
      _object_space = new MutableSpace(virtual_space()->alignment());
```

* MutableNUMASpace::LGRPSpace::LGRPSpace()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableNUMASpace.hpp))
        LGRPSpace(int l, size_t alignment) : _lgrp_id(l), _last_page_scanned(NULL), _allocation_failed(false) {
          _space = new MutableSpace(alignment);
```

### 内部構造(Internal structure)
ImmutableSpace クラスの機能に加えて, 対象の領域内からメモリを確保する機能が追加されている.
そのために, 使用済みの範囲を示す _top フィールドが追加されている
(これは ContiguousSpace クラスの同名のフィールドとほぼ同じ.
 _bottom から _top までが使用済みの範囲, _top から _end までが未使用の範囲).

またその他に, メモリページの確保を高速化するためにプレタッチしておく機能 (See: AlwaysPretouch) や
NUMA 上での効率を考えてページの配置を最適化する機能 (See: UseNUMA) 等も備えている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/mutableSpace.hpp))
    // A MutableSpace is a subtype of ImmutableSpace that supports the
    // concept of allocation. This includes the concepts that a space may
    // be only partially full, and the querry methods that go with such
    // an assumption. MutableSpace is also responsible for minimizing the
    // page allocation time by having the memory pretouched (with
    // AlwaysPretouch) and for optimizing page placement on NUMA systems
    // by make the underlying region interleaved (with UseNUMA).
    //
    // Invariant: (ImmutableSpace +) bottom() <= top() <= end()
    // top() is inclusive and end() is exclusive.
```




### 詳細(Details)
See: [here](../doxygen/classMutableSpace.html) for details

---
