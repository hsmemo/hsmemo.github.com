---
layout: default
title: GCAdaptivePolicyCounters クラス 
---
[Top](../index.html)

#### GCAdaptivePolicyCounters クラス 



---
## <a name="nostX2hDy8" id="nostX2hDy8">GCAdaptivePolicyCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

AdaptiveSizePolicy に関する PerfData を格納しておくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcAdaptivePolicyCounters.hpp))
    // This class keeps statistical information and computes the
    // size of the heap.
    
    class GCAdaptivePolicyCounters : public GCPolicyCounters {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 内部構造(Internal structure)
以下のような情報を格納している.

  * sun.gc.policy.edenSize
  * sun.gc.policy.promoSize
  * sun.gc.policy.youngCapacity
  * sun.gc.policy.avgSurvivedAvg
  * sun.gc.policy.avgSurvivedDev
  * sun.gc.policy.avgSurvivedPaddedAvg
  * sun.gc.policy.avgMinorPauseTime
  * sun.gc.policy.avgMinorIntervalTime
  * sun.gc.policy.minorPauseTime
  * sun.gc.policy.minorGcCost
  * sun.gc.policy.mutatorCost
  * sun.gc.policy.survived
  * sun.gc.policy.promoted
  * sun.gc.policy.avgYoungLive
  * sun.gc.policy.avgOldLive
  * sun.gc.policy.survivorOverflowed
  * sun.gc.policy.decrementTenuringThresholdForGcCost
  * sun.gc.policy.incrementTenuringThresholdForGcCost
  * sun.gc.policy.decrementTenuringThresholdForSurvivorLimit
  * sun.gc.policy.changeYoungGenForMinPauses
  * sun.gc.policy.changeOldGenForMajPauses
  * sun.gc.policy.increaseOldGenForThroughput
  * sun.gc.policy.increaseYoungGenForThroughput
  * sun.gc.policy.decreaseForFootprint
  * sun.gc.policy.decideAtFullGc
  * sun.gc.policy.minorPauseYoungSlope
  * sun.gc.policy.majorCollectionSlope
  * sun.gc.policy.minorCollectionSlope

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/gcAdaptivePolicyCounters.cpp))
        const char* cname = PerfDataManager::counter_name(name_space(), "edenSize");
        _eden_size_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, _size_policy->calculated_eden_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "promoSize");
        _promo_size_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, size_policy()->calculated_promo_size_in_bytes(),
          CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "youngCapacity");
        size_t young_capacity_in_bytes =
          _size_policy->calculated_eden_size_in_bytes() +
          _size_policy->calculated_survivor_size_in_bytes();
        _young_capacity_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, young_capacity_in_bytes, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgSurvivedAvg");
        _avg_survived_avg_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, size_policy()->calculated_survivor_size_in_bytes(),
            CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgSurvivedDev");
        _avg_survived_dev_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0 , CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgSurvivedPaddedAvg");
        _avg_survived_padded_avg_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
            size_policy()->calculated_survivor_size_in_bytes(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgMinorPauseTime");
        _avg_minor_pause_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Ticks, (jlong) _size_policy->_avg_minor_pause->average(),
          CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgMinorIntervalTime");
        _avg_minor_interval_counter = PerfDataManager::create_variable(SUN_GC,
          cname,
          PerfData::U_Ticks,
          (jlong) _size_policy->_avg_minor_interval->average(),
          CHECK);
    
    #ifdef NOT_PRODUCT
          // This is a counter for the most recent minor pause time
          // (the last sample, not the average).  It is useful for
          // verifying the average pause time but not worth putting
          // into the product.
          cname = PerfDataManager::counter_name(name_space(), "minorPauseTime");
          _minor_pause_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Ticks, (jlong) _size_policy->_avg_minor_pause->last_sample(),
          CHECK);
    #endif
    
        cname = PerfDataManager::counter_name(name_space(), "minorGcCost");
        _minor_gc_cost_counter = PerfDataManager::create_variable(SUN_GC,
          cname,
          PerfData::U_Ticks,
          (jlong) _size_policy->minor_gc_cost(),
          CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "mutatorCost");
        _mutator_cost_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Ticks, (jlong) _size_policy->mutator_cost(), CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "survived");
        _survived_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "promoted");
        _promoted_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgYoungLive");
        _avg_young_live_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) size_policy()->avg_young_live()->average(),
          CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "avgOldLive");
        _avg_old_live_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Bytes, (jlong) size_policy()->avg_old_live()->average(),
          CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "survivorOverflowed");
        _survivor_overflowed_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Events, (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "decrementTenuringThresholdForGcCost");
        _decrement_tenuring_threshold_for_gc_cost_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "incrementTenuringThresholdForGcCost");
        _increment_tenuring_threshold_for_gc_cost_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "decrementTenuringThresholdForSurvivorLimit");
        _decrement_tenuring_threshold_for_survivor_limit_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
        cname = PerfDataManager::counter_name(name_space(),
          "changeYoungGenForMinPauses");
        _change_young_gen_for_min_pauses_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "changeOldGenForMajPauses");
        _change_old_gen_for_maj_pauses_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "increaseOldGenForThroughput");
        _change_old_gen_for_throughput_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "increaseYoungGenForThroughput");
        _change_young_gen_for_throughput_counter =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Events,
            (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(),
          "decreaseForFootprint");
        _decrease_for_footprint_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_Events, (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "decideAtFullGc");
        _decide_at_full_gc_counter = PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong)0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "minorPauseYoungSlope");
        _minor_pause_young_slope_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "majorCollectionSlope");
        _major_collection_slope_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(name_space(), "minorCollectionSlope");
        _minor_collection_slope_counter =
          PerfDataManager::create_variable(SUN_GC, cname,
          PerfData::U_None, (jlong) 0, CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classGCAdaptivePolicyCounters.html) for details

---
