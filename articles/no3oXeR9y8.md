---
layout: default
title: ASPSOldGen クラス 
---
[Top](../index.html)

#### ASPSOldGen クラス 



---
## <a name="noSGEpCAJj" id="noSGEpCAJj">ASPSOldGen</a>

### 概要(Summary)
ParallelScavengeHeap 使用時において, Old Generation の管理を担当するクラスの 1つ (See: [here](no3718kvd.html) for details).

このクラスは, UseAdaptiveGCBoundary オプションが指定されている場合用 
(= GC Ergonomics を用いた動的領域サイズ調整を行う場合用) (See: PSOldGen).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/asPSOldGen.hpp))
    class ASPSOldGen : public PSOldGen {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap オブジェクトの _old_gen フィールドに格納されている.

(ただしこのフィールドには, ASPSOldGen オブジェクトではなく,
スーパークラスである PSOldGen オブジェクトが格納されることもある.
その場合にはこのクラスはどこからも使用されない.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      static PSOldGen*   _old_gen;
```

(なお, AdjoiningGenerations オブジェクトの _old_gen フィールドにも同じインスタンスが格納されている.
正確には, こちらで生成されたものが ParallelScavengeHeap のフィールドにコピーされる)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.hpp))
    class AdjoiningGenerations : public CHeapObj {
    ...
      PSOldGen* _old_gen;
```

#### 生成箇所(where its instances are created)
AdjoiningGenerations::AdjoiningGenerations() 内で(のみ)生成されている.

(なお ASPSOldGen オブジェクトが生成されるかどうかは UseAdaptiveGCBoundary オプションの値によって決まる.
UseAdaptiveGCBoundary オプションが指定されている場合は ASPSOldGen が生成され, そうでない場合は PSOldGen が生成される)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.cpp))
    AdjoiningGenerations::AdjoiningGenerations(ReservedSpace old_young_rs,
                                               size_t init_low_byte_size,
                                               size_t min_low_byte_size,
                                               size_t max_low_byte_size,
                                               size_t init_high_byte_size,
                                               size_t min_high_byte_size,
                                               size_t max_high_byte_size,
                                               size_t alignment) :
    ...
      if (UseAdaptiveGCBoundary) {
    ...
        // Place the old gen at the low end. Passes in the virtual space.
        _old_gen = new ASPSOldGen(_virtual_spaces.low(),
                                  _virtual_spaces.low()->committed_size(),
                                  min_low_byte_size,
                                  _virtual_spaces.low_byte_size_limit(),
                                  "old", 1);
    ...
      } else {
    ...
        _old_gen = new PSOldGen(init_low_byte_size,
                                min_low_byte_size,
                                max_low_byte_size,
                                "old", 1);
```




### 詳細(Details)
See: [here](../doxygen/classASPSOldGen.html) for details

---
