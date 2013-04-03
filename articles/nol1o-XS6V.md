---
layout: default
title: PSGCAdaptivePolicyCounters クラス 
---
[Top](../index.html)

#### PSGCAdaptivePolicyCounters クラス 



---
## <a name="norQH1mHTf" id="norQH1mHTf">PSGCAdaptivePolicyCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

ParallelScavenge 用の GCAdaptivePolicyCounters クラス
(つまり, AdaptiveSizePolicy に関する PerfData を格納しておくためのクラス (See: GCAdaptivePolicyCounters)).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psGCAdaptivePolicyCounters.hpp))
    // PSGCAdaptivePolicyCounters is a holder class for performance counters
    // that track the data and decisions for the ergonomics policy for the
    // parallel scavenge collector.
    
    class PSGCAdaptivePolicyCounters : public GCAdaptivePolicyCounters {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ParallelScavengeHeap クラスの _gc_policy_counters フィールドに(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.hpp))
    class ParallelScavengeHeap : public CollectedHeap {
    ...
      static PSGCAdaptivePolicyCounters*   _gc_policy_counters;
```

### 内部構造(Internal structure)
GCAdaptivePolicyCounters が格納している情報に加えて, 
以下のような情報を格納している.

  * sun.gc.policy.oldPromoSize
  * sun.gc.policy.oldEdenSize
  * sun.gc.policy.oldCapacity
  * sun.gc.policy.boundaryMoved
  * sun.gc.policy.avgPromotedAvg
  * sun.gc.policy.avgPromotedDev
  * sun.gc.policy.avgPromotedPaddedAvg
  * sun.gc.policy.avgPretenuredPaddedAvg
  * sun.gc.policy.changeYoungGenForMajPauses
  * sun.gc.policy.changeOldGenForMinPauses
  * sun.gc.policy.avgMajorPauseTime
  * sun.gc.policy.avgMajorIntervalTime
  * sun.gc.policy.majorGcCost
  * sun.gc.policy.liveSpace
  * sun.gc.policy.freeSpace
  * sun.gc.policy.avgBaseFootprint
  * sun.gc.policy.gcTimeLimitExceeded
  * sun.gc.policy.liveAtLastFullGc
  * sun.gc.policy.majorPauseOldSlope
  * sun.gc.policy.minorPauseOldSlope
  * sun.gc.policy.majorPauseYoungSlope
  * sun.gc.policy.scavengeSkipped
  * sun.gc.policy.fullFollowsScavenge


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psGCAdaptivePolicyCounters.cpp))
    PSGCAdaptivePolicyCounters::PSGCAdaptivePolicyCounters(const char* name_arg,
                                          int collectors,
                                          int generations,
                                          PSAdaptiveSizePolicy* size_policy_arg)
            : GCAdaptivePolicyCounters(name_arg,
                                       collectors,
                                       generations,
                                       size_policy_arg) {
      if (UsePerfData) {
        EXCEPTION_MARK;
        ResourceMark rm;
    
        const char* cname;
    
        cname = PerfDataManager::counter_name(name_space(), "oldPromoSize");
        _old_promo_size = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, ps_size_policy()->calculated_promo_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "oldEdenSize");
        _old_eden_size = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, ps_size_policy()->calculated_eden_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "oldCapacity");
        _old_capacity = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) InitialHeapSize, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "boundaryMoved");
        _boundary_moved = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgPromotedAvg");
        _avg_promoted_avg_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
            ps_size_policy()->calculated_promo_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgPromotedDev");
        _avg_promoted_dev_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
            (jlong) 0 , CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgPromotedPaddedAvg");
        _avg_promoted_padded_avg_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
            ps_size_policy()->calculated_promo_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "avgPretenuredPaddedAvg");
        _avg_pretenured_padded_avg =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
            (jlong) 0, CHECK);
    
    
        cname = PerfDataManager::counter_name(name_space(),
          "changeYoungGenForMajPauses");
        _change_young_gen_for_maj_pauses_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "changeOldGenForMinPauses");
        _change_old_gen_for_min_pauses =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
    
        cname = PerfDataManager::counter_name(name_space(), "avgMajorPauseTime");
        _avg_major_pause = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Ticks, (jlong) ps_size_policy()->_avg_major_pause->average(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgMajorIntervalTime");
        _avg_major_interval = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Ticks, (jlong) ps_size_policy()->_avg_major_interval->average(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "majorGcCost");
        _major_gc_cost_counter = PerfDataManager::create_variable(SUN_GC, cname,
           PerfData::U_Ticks, (jlong) ps_size_policy()->major_gc_cost(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "liveSpace");
        _live_space = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, ps_size_policy()->live_space(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "freeSpace");
        _free_space = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, ps_size_policy()->free_space(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgBaseFootprint");
        _avg_base_footprint = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) ps_size_policy()->avg_base_footprint()->average(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "gcTimeLimitExceeded");
        _gc_overhead_limit_exceeded_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Events, ps_size_policy()->gc_overhead_limit_exceeded(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "liveAtLastFullGc");
        _live_at_last_full_gc_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, ps_size_policy()->live_at_last_full_gc(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "majorPauseOldSlope");
        _major_pause_old_slope = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "minorPauseOldSlope");
        _minor_pause_old_slope = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "majorPauseYoungSlope");
        _major_pause_young_slope = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "scavengeSkipped");
        _scavenge_skipped = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "fullFollowsScavenge");
        _full_follows_scavenge = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0, CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classPSGCAdaptivePolicyCounters.html) for details

---
