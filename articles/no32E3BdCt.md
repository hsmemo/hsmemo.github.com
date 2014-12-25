---
layout: default
title: GCPolicyCounters クラス 
---
[Top](../index.html)

#### GCPolicyCounters クラス 



---
## <a name="nocfbz70RV" id="nocfbz70RV">GCPolicyCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

CollectorPolicy(?) に関する PerfData を格納しておくためのクラス.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcPolicyCounters.hpp))
    // GCPolicyCounters is a holder class for performance counters
    // that track a generation
    
    class GCPolicyCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CollectorPolicy オブジェクトの _gc_policy_counters フィールドに(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    class CollectorPolicy : public CHeapObj {
    ...
      GCPolicyCounters* _gc_policy_counters;
```

(ただし, このフィールドには GCPolicyCounters クラスではなく, そのサブクラスのオブジェクトが格納されることもある (See: CMSGCAdaptivePolicyCounters, PSGCAdaptivePolicyCounters))

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ConcurrentMarkSweepPolicy::initialize_gc_policy_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/cmsCollectorPolicy.cpp))
    void ConcurrentMarkSweepPolicy::initialize_gc_policy_counters() {
      // initialize the policy counters - 2 collectors, 3 generations
      if (ParNewGeneration::in_use()) {
        _gc_policy_counters = new GCPolicyCounters("ParNew:CMS", 2, 3);
      }
      else {
        _gc_policy_counters = new GCPolicyCounters("Copy:CMS", 2, 3);
      }
```

* G1CollectorPolicy::initialize_gc_policy_counters()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp))
    void G1CollectorPolicy::initialize_gc_policy_counters()
    {
      _gc_policy_counters = new GCPolicyCounters("GarbageFirst", 1, 2 + G1Gen);
```

* MarkSweepPolicy::initialize_gc_policy_counters()

```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.cpp))
    void MarkSweepPolicy::initialize_gc_policy_counters() {
      // initialize the policy counters - 2 collectors, 3 generations
      if (UseParNewGC && ParallelGCThreads > 0) {
        _gc_policy_counters = new GCPolicyCounters("ParNew:MSC", 2, 3);
      }
      else {
        _gc_policy_counters = new GCPolicyCounters("Copy:MSC", 2, 3);
      }
```

### 内部構造(Internal structure)
以下のような情報を格納している.

(なお, sun.gc.policy.name に対しては, コンストラクタに渡された文字列が格納されている.
 例えば, ParallelScavenge の場合なら "ParScav:MSC", CMS の場合なら "ParNew:CMS" や "Copy:CMS" 等.
 (See: PSGCAdaptivePolicyCounters, CMSGCAdaptivePolicyCounters))

  * sun.gc.policy.name
  * sun.gc.policy.collectors
  * sun.gc.policy.generations
  * sun.gc.policy.maxTenuringThreshold
  * sun.gc.policy.tenuringThreshold
  * sun.gc.policy.desiredSurvivorSize

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcPolicyCounters.cpp))
        _name_space = "policy";
    
        char* cname = PerfDataManager::counter_name(_name_space, "name");
        PerfDataManager::create_string_constant(SUN_GC, cname, name, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "collectors");
        PerfDataManager::create_constant(SUN_GC, cname,  PerfData::U_None,
                                         collectors, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "generations");
        PerfDataManager::create_constant(SUN_GC, cname,  PerfData::U_None,
                                         generations, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "maxTenuringThreshold");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_None,
                                         MaxTenuringThreshold, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "tenuringThreshold");
        _tenuring_threshold =
            PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None,
                                             MaxTenuringThreshold, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "desiredSurvivorSize");
        _desired_survivor_size =
            PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                             CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classGCPolicyCounters.html) for details

---
