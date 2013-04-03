---
layout: default
title: GCCause クラス 
---
[Top](../index.html)

#### GCCause クラス 



---
## <a name="noqMjsIB6M" id="noqMjsIB6M">GCCause</a>

### 概要(Summary)
HotSpot 内部で起こりうる GC の発生原因に対して, それを一意に示す定数値を定義した名前空間(AllStatic クラス).


```
    ((cite: hotspot/src/share/vm/gc_interface/gcCause.hpp))
    //
    // This class exposes implementation details of the various
    // collector(s), and we need to be very careful with it. If
    // use of this class grows, we should split it into public
    // and implemenation-private "causes".
    //
    
    class GCCause : public AllStatic {
```

### 内部構造(Internal structure)
定義されている「GC 発生原因」の一覧は以下の通り.


```
    ((cite: hotspot/src/share/vm/gc_interface/gcCause.hpp))
      enum Cause {
        /* public */
        _java_lang_system_gc,
        _full_gc_alot,
        _scavenge_alot,
        _allocation_profiler,
        _jvmti_force_gc,
        _gc_locker,
        _heap_inspection,
        _heap_dump,
    
        /* implementation independent, but reserved for GC use */
        _no_gc,
        _no_cause_specified,
        _allocation_failure,
    
        /* implementation specific */
    
        _tenured_generation_full,
        _permanent_generation_full,
    
        _cms_generation_full,
        _cms_initial_mark,
        _cms_final_remark,
    
        _old_generation_expanded_on_last_scavenge,
        _old_generation_too_full_to_scavenge,
        _adaptive_size_policy,
    
        _g1_inc_collection_pause,
    
        _last_ditch_collection,
        _last_gc_cause
      };
```




### 詳細(Details)
See: [here](../doxygen/classGCCause.html) for details

---
