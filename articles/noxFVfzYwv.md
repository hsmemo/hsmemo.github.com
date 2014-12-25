---
layout: default
title: GenerationCounters クラス 
---
[Top](../index.html)

#### GenerationCounters クラス 



---
## <a name="no6NTJeHK1" id="no6NTJeHK1">GenerationCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

Generation に関する PerfData を格納しておくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/generationCounters.hpp))
    // A GenerationCounter is a holder class for performance counters
    // that track a generation
    
    class GenerationCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Generation クラスの各種のサブクラス内に保持されている.
また, Generation クラスのサブクラスを使っていない G1GC の場合にも
G1MonitoringSupport クラス内に保持されている.

* DefNewGeneration

```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.hpp))
    class DefNewGeneration: public Generation {
    ...
      // Performance Counters
      GenerationCounters*  _gen_counters;
```

* TenuredGeneration

```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.hpp))
    class TenuredGeneration: public OneContigSpaceCardGeneration {
    ...
      GenerationCounters*   _gen_counters;
```

* CompactingPermGenGen

```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.hpp))
    class CompactingPermGenGen: public OneContigSpaceCardGeneration {
    ...
      // Performance Counters
      GenerationCounters*  _gen_counters;
```

* ConcurrentMarkSweepGeneration

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.hpp))
    class ConcurrentMarkSweepGeneration: public CardGeneration {
    ...
      // Performance Counters
      GenerationCounters*      _gen_counters;
```

* G1MonitoringSupport

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp))
    class G1MonitoringSupport : public CHeapObj {
    ...
      //  young collection set counters.  The _eden_counters,
      // _from_counters, and _to_counters are associated with
      // this "generational" counter.
      GenerationCounters*  _young_collection_counters;
      //  non-young collection set counters. The _old_space_counters
      // below are associated with this "generational" counter.
      GenerationCounters*  _non_young_collection_counters;
```

#### 生成箇所(where its instances are created)
それぞれ, 以下の箇所でインスタンスが生成されている.

* DefNewGeneration::DefNewGeneration()

```cpp
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    DefNewGeneration::DefNewGeneration(ReservedSpace rs,
                                       size_t initial_size,
                                       int level,
                                       const char* policy)
    ...
    {
    ...
      // Generation counters -- generation 0, 3 subspaces
      _gen_counters = new GenerationCounters("new", 0, 3, &_virtual_space);
```

* TenuredGeneration::TenuredGeneration()

```cpp
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.cpp))
    TenuredGeneration::TenuredGeneration(ReservedSpace rs,
                                         size_t initial_byte_size, int level,
                                         GenRemSet* remset) :
    ...
    {
    ...
      // Generation Counters -- generation 1, 1 subspace
      _gen_counters = new GenerationCounters(gen_name, 1, 1, &_virtual_space);
```

* CompactingPermGenGen::initialize_performance_counters()

```cpp
    ((cite: hotspot/src/share/vm/memory/compactingPermGenGen.cpp))
    void CompactingPermGenGen::initialize_performance_counters() {
    ...
      // Generation Counters - generation 2, 1 subspace
      _gen_counters = new GenerationCounters(gen_name, 2, 1, &_virtual_space);
```

* CMSPermGenGen::initialize_performance_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/cmsPermGen.cpp))
    void CMSPermGenGen::initialize_performance_counters() {
    ...
      // Generation Counters - generation 2, 1 subspace
      _gen_counters = new GenerationCounters(gen_name, 2, 1, &_virtual_space);
```

* ConcurrentMarkSweepGeneration::initialize_performance_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp))
    void ConcurrentMarkSweepGeneration::initialize_performance_counters() {
    ...
      // Generation Counters - generation 1, 1 subspace
      _gen_counters = new GenerationCounters(gen_name, 1, 1, &_virtual_space);
```

* G1MonitoringSupport::G1MonitoringSupport()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.cpp))
    G1MonitoringSupport::G1MonitoringSupport(G1CollectedHeap* g1h,
                                             VirtualSpace* g1_storage_addr) :
    ...
    {
    ...
      // "Generation" and "Space" counters.
      //
      //  name "generation.1" This is logically the old generation in
      // generational GC terms.  The "1, 1" parameters are for
      // the n-th generation (=1) with 1 space.
      // Counters are created from minCapacity, maxCapacity, and capacity
      _non_young_collection_counters =
        new GenerationCounters("whole heap", 1, 1, _g1_storage_addr);
    ...
      //   Young collection set
      //  name "generation.0".  This is logically the young generation.
      //  The "0, 3" are paremeters for the n-th genertaion (=0) with 3 spaces.
      // See  _non_young_collection_counters for additional counters
      _young_collection_counters = new GenerationCounters("young", 0, 3, NULL);
```

### 内部構造(Internal structure)
各 Generation の以下の情報が格納されている.
(現在の領域長だけは実行時に変わるので GC の後などに変更している. それ以外は生成時に指定した値で固定.)

* 世代名("name")
* 領域数(?)("spaces"), 
* 最小領域長("minCapacity"), 
* 最大領域長("maxCapacity"), 
* 現在の領域長("capacity")

それぞれ以下の名前でアクセス可能 
(${n} の箇所には 0, 1, ... といった数字が入る. 例えば sun.gc.generation.0.name 等)

  * sun.gc.generation.${n}.name
  * sun.gc.generation.${n}.spaces
  * sun.gc.generation.${n}.minCapacity
  * sun.gc.generation.${n}.maxCapacity
  * sun.gc.generation.${n}.capacity

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/generationCounters.cpp))
        const char* cns = PerfDataManager::name_space("generation", ordinal);
    
        _name_space = NEW_C_HEAP_ARRAY(char, strlen(cns)+1);
        strcpy(_name_space, cns);
    
        const char* cname = PerfDataManager::counter_name(_name_space, "name");
        PerfDataManager::create_string_constant(SUN_GC, cname, name, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "spaces");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_None,
                                         spaces, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "minCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         _virtual_space == NULL ? 0 :
                                         _virtual_space->committed_size(), CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "maxCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         _virtual_space == NULL ? 0 :
                                         _virtual_space->reserved_size(), CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "capacity");
        _current_size = PerfDataManager::create_variable(SUN_GC, cname,
                                         PerfData::U_Bytes,
                                         _virtual_space == NULL ? 0 :
                                         _virtual_space->committed_size(), CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classGenerationCounters.html) for details

---
