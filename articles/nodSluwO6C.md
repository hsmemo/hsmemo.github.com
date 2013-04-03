---
layout: default
title: PSOldGen クラス (PSOldGen, 及びその補助クラス(VerifyObjectStartArrayClosure))
---
[Top](../index.html)

#### PSOldGen クラス (PSOldGen, 及びその補助クラス(VerifyObjectStartArrayClosure))

これらは, ParallelScavengeHeap 使用時において, Old Generation の管理を担当するクラス (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [PSOldGen](#noCy_YQ1os)
  * [VerifyObjectStartArrayClosure](#noCPROy2cg)


---
## <a name="noCy_YQ1os" id="noCy_YQ1os">PSOldGen</a>

### 概要(Summary)
ParallelScavengeHeap 使用時において, Old Generation の管理を担当するクラスの 1つ (See: [here](no3718kvd.html) for details).

このクラスは, UseAdaptiveGCBoundary オプションが指定されていない場合用 
(= GC Ergonomics を用いた動的領域サイズ調整を行わない場合用) (See: ASPSOldGen).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.hpp))
    class PSOldGen : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap オブジェクトの _old_gen フィールドに格納されている.

(ただしこのフィールドには, PSOldGen オブジェクトではなく,
サブクラスである ASPSOldGen オブジェクトが格納されることもある.
その場合にはこのクラスはどこからも使用されない.)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      static PSOldGen*   _old_gen;
```

(なお, AdjoiningGenerations オブジェクトの _old_gen フィールドにも同じインスタンスが格納されている.
正確には, こちらで生成されたものが ParallelScavengeHeap のフィールドにコピーされる)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningGenerations.hpp))
    class AdjoiningGenerations : public CHeapObj {
    ...
      PSOldGen* _old_gen;
```

#### 生成箇所(where its instances are created)
AdjoiningGenerations::AdjoiningGenerations() 内で(のみ)生成されている.

(なお PSOldGen オブジェクトが生成されるかどうかは UseAdaptiveGCBoundary オプションの値によって決まる.
UseAdaptiveGCBoundary オプションが指定されている場合は ASPSOldGen が生成され, そうでない場合は PSOldGen が生成される)


```
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
See: [here](../doxygen/classPSOldGen.html) for details

---
## <a name="noCPROy2cg" id="noCPROy2cg">VerifyObjectStartArrayClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される)
(See: VerifyObjectStartArray, VerifyBeforeGC, VerifyAfterGC).

PSOldGen::verify_object_start_array() 内で使用される補助クラス.
PSOldGen オブジェクト内にある ObjectStartArray のチェックを行う.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.cpp))
    class VerifyObjectStartArrayClosure : public ObjectClosure {
```

### 使われ方(Usage)
PSOldGen::verify_object_start_array() 内で(のみ)使用されている.

(なお, PSOldGen::verify_object_start_array() は以下の関数から呼び出される verify 用の関数.
 ただし, VerifyObjectStartArray, VerifyBeforeGC, VerifyAfterGC
 オプションが指定されている場合にしか呼び出されない.

 * VerifyObjectStartArray と VerifyBeforeGC が両方指定されている場合
   * PSMarkSweep::invoke_no_policy()
   * PSParallelCompact::pre_compact()
   * PSScavenge::invoke_no_policy()

 * VerifyObjectStartArray と VerifyAfterGC が両方指定されている場合
   * PSMarkSweep::invoke_no_policy()
   * PSParallelCompact::invoke_no_policy()
   * PSScavenge::invoke_no_policy())




### 詳細(Details)
See: [here](../doxygen/classVerifyObjectStartArrayClosure.html) for details

---
