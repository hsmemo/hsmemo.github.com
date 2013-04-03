---
layout: default
title: GenerationSizer クラス 
---
[Top](../index.html)

#### GenerationSizer クラス 



---
## <a name="no6A0UOqwZ" id="no6A0UOqwZ">GenerationSizer</a>

### 概要(Summary)
ParallelScavenge 用の CollectorPolicy クラス.

(TwoGenerationCollectorPolicy クラスの具象サブクラスの1つ (See: [here](no3718kvd.html) for details))


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/generationSizer.hpp))
    // There is a nice batch of tested generation sizing code in
    // TwoGenerationCollectorPolicy. Lets reuse it!
    
    class GenerationSizer : public TwoGenerationCollectorPolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap クラスの _collector_policy フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      GenerationSizer* _collector_policy;
```

#### 生成箇所(where its instances are created)
ParallelScavengeHeap::initialize() 内で(のみ)生成されている.

#### 参考(for your information): ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details



### 詳細(Details)
See: [here](../doxygen/classGenerationSizer.html) for details

---
