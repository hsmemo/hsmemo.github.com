---
layout: default
title: PSGenerationCounters クラス 
---
[Top](../index.html)

#### PSGenerationCounters クラス 



---
## <a name="noI13mdHJe" id="noI13mdHJe">PSGenerationCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

ParallelScavenge 用の GenerationCounters
(つまり, Generation に関する PerfData を格納しておくためのクラス (See: GenerationCounters)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psGenerationCounters.hpp))
    // A PSGenerationCounter is a holder class for performance counters
    // that track a generation
    
    class PSGenerationCounters: public GenerationCounters {
```

なお, わざわざ GenerationCounters のサブクラスを作っているのは ParallelScavenge だけ.
何でこうなっているかと言えば, VirtualSpace と PSVirtualSpace が互換性がない(サブタイプ関係にない)ため.
(将来的に VirtualSpace と PSVirtualSpace が統合されれば 
PSGenerationCounters も GenerationCounters に統合されるかも)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/generationCounters.hpp))
      // This constructor is only meant for use with the PSGenerationCounters
      // constructor.  The need for such an constructor should be eliminated
      // when VirtualSpace and PSVirtualSpace are unified.
      GenerationCounters() : _name_space(NULL), _current_size(NULL), _virtual_space(NULL) {}
```




### 詳細(Details)
See: [here](../doxygen/classPSGenerationCounters.html) for details

---
