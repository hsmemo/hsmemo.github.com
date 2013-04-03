---
layout: default
title: HSpaceCounters クラス 
---
[Top](../index.html)

#### HSpaceCounters クラス 



---
## <a name="nollB_0jM4" id="nollB_0jM4">HSpaceCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

G1GC で使用される「論理的な Space」に関する PerfData を格納しておくためのクラス
(See: [here](no3718kvd.html) for details) (See: PerfData).

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.hpp))
    // A HSpaceCounter is a holder class for performance counters
    // that track a collections (logical spaces) in a heap;
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.hpp))
    class HSpaceCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 G1MonitoringSupport オブジェクトの _old_space_counters フィールド
* 各 G1MonitoringSupport オブジェクトの _eden_counters フィールド
* 各 G1MonitoringSupport オブジェクトの _from_counters フィールド
* 各 G1MonitoringSupport オブジェクトの _to_counters フィールド


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp))
    class G1MonitoringSupport : public CHeapObj {
    ...
      // Counters for the capacity and used for
      //   the whole heap
      HSpaceCounters*      _old_space_counters;
      //   the young collection
      HSpaceCounters*      _eden_counters;
      //   the survivor collection (only one, _to_counters, is actively used)
      HSpaceCounters*      _from_counters;
      HSpaceCounters*      _to_counters;
```

#### 生成箇所(where its instances are created)
G1MonitoringSupport::G1MonitoringSupport() 内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.cpp))
    G1MonitoringSupport::G1MonitoringSupport(G1CollectedHeap* g1h,
                                             VirtualSpace* g1_storage_addr) :
    ...
    {
    ...
      //  name  "generation.1.space.0"
      // Counters are created from maxCapacity, capacity, initCapacity,
      // and used.
      _old_space_counters = new HSpaceCounters("space", 0,
        _g1h->max_capacity(), _g1h->capacity(), _non_young_collection_counters);
    ...
    
      // Replace "max_heap_byte_size() with maximum young gen size for
      // g1Collectedheap
      //  name "generation.0.space.0"
      // See _old_space_counters for additional counters
      _eden_counters = new HSpaceCounters("eden", 0,
        _g1h->max_capacity(), eden_space_committed(),
        _young_collection_counters);
    
      //  name "generation.0.space.1"
      // See _old_space_counters for additional counters
      // Set the arguments to indicate that this survivor space is not used.
      _from_counters = new HSpaceCounters("s0", 1, (long) 0, (long) 0,
        _young_collection_counters);
    
      //  name "generation.0.space.2"
      // See _old_space_counters for additional counters
      _to_counters = new HSpaceCounters("s1", 2,
        _g1h->max_capacity(),
        survivor_space_committed(),
        _young_collection_counters);
```

### 内部構造(Internal structure)
内部には, パフォーマンスカウンタとして使う PerfVariable 2 個を保持している.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.hpp))
      PerfVariable*        _capacity;
      PerfVariable*        _used;
```

これらの PerfVariable には, 対応するアクセサ関数に渡される値が記録される.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.hpp))
      inline void update_capacity(size_t v) {
        _capacity->set_value(v);
      }
    
      inline void update_used(size_t v) {
        _used->set_value(v);
      }
```

正確には, 可変でない項目も入れると, 各論理Space の以下の情報が格納されている
(現在の領域長や現在の使用量は実行時に変わるので GC 後などに変更している. それ以外は生成時に指定した値で固定).

* 領域名("name")
* 最大領域長("maxCapacity")
* 現在の領域長("capacity")
* 現在の使用量("used")
* 初期領域長("initCapacity") 

それぞれ以下の名前でアクセス可能 
(${n} や ${m} の箇所には 0, 1, ... といった数字が入る. 例えば sun.gc.generation.0.space.1.name 等)

  * sun.gc.generation.${n}.space.${m}.name
  * sun.gc.generation.${n}.space.${m}.maxCapacity
  * sun.gc.generation.${n}.space.${m}.capacity
  * sun.gc.generation.${n}.space.${m}.used
  * sun.gc.generation.${n}.space.${m}.initCapacity

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/hSpaceCounters.cpp))
        const char* cns =
          PerfDataManager::name_space(gc->name_space(), "space", ordinal);
    
        _name_space = NEW_C_HEAP_ARRAY(char, strlen(cns)+1);
        strcpy(_name_space, cns);
    
        const char* cname = PerfDataManager::counter_name(_name_space, "name");
        PerfDataManager::create_string_constant(SUN_GC, cname, name, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "maxCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         (jlong)max_size, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "capacity");
        _capacity = PerfDataManager::create_variable(SUN_GC, cname,
                                                     PerfData::U_Bytes,
                                                     initial_capacity, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "used");
        _used = PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes,
                                                 (jlong) 0, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "initCapacity");
        PerfDataManager::create_constant(SUN_GC, cname, PerfData::U_Bytes,
                                         initial_capacity, CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classHSpaceCounters.html) for details

---
