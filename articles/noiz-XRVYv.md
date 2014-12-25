---
layout: default
title: PSPermGen クラス 
---
[Top](../index.html)

#### PSPermGen クラス 



---
## <a name="noAtiTcrwQ" id="noAtiTcrwQ">PSPermGen</a>

### 概要(Summary)
ParallelScavengeHeap 使用時において, Perm Generation の管理を担当するクラス (See: [here](no3718kvd.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psPermGen.hpp))
    class PSPermGen : public PSOldGen {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap オブジェクトの _perm_gen フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      static PSPermGen*  _perm_gen;
```

#### 生成箇所(where its instances are created)
ParallelScavengeHeap::initialize() 内で(のみ)生成されている.

#### 参考(for your information): ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSPermGen.html) for details

---
