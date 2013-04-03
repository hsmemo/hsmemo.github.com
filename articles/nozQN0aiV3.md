---
layout: default
title: CollectorCounters クラス関連のクラス (CollectorCounters, TraceCollectorStats)
---
[Top](../index.html)

#### CollectorCounters クラス関連のクラス (CollectorCounters, TraceCollectorStats)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, Garbage Collection 処理に関する情報を PerfData 形式で出力するためのクラス
(See: [here](no3718kvd.html) for details) (See: PerfData).


### クラス一覧(class list)

  * [CollectorCounters](#noZFpB5gT9)
  * [TraceCollectorStats](#noANAamRFG)


---
## <a name="noZFpB5gT9" id="noZFpB5gT9">CollectorCounters</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス).

GC 処理に関する PerfData を格納しておくためのクラス.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp))
    // CollectorCounters is a holder class for performance counters
    // that track a collector
    
    class CollectorCounters: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各種の GC 処理を担当するクラス内に格納されている.

* Generation 及びそのサブクラス (DefNewGeneration, ParNewGeneration, ASParNewGeneration, TenuredGeneration)

```
    ((cite: hotspot/src/share/vm/memory/generation.hpp))
    class Generation: public CHeapObj {
    ...
      // Performance Counters
      CollectorCounters* _gc_counters;
```

* CMSCollector

```
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.hpp))
    class CMSCollector: public CHeapObj {
    ...
      // Performance Counters
      CollectorCounters* _gc_counters;
```

* PSScavenge

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.hpp))
    class PSScavenge: AllStatic {
    ...
      static CollectorCounters*      _counters;         // collector performance counters
```

* PSMarkSweep

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.hpp))
    class PSMarkSweep : public MarkSweep {
    ...
      static CollectorCounters*  _counters;
```

* PSParallelCompact

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp))
    class PSParallelCompact : AllStatic {
    ...
      static CollectorCounters*   _counters;
```

* G1MonitoringSupport

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.hpp))
    class G1MonitoringSupport : public CHeapObj {
    ...
      // jstat performance counters
      //  incremental collections both fully and partially young
      CollectorCounters*   _incremental_collection_counters;
      //  full stop-the-world collections
      CollectorCounters*   _full_collection_counters;
```


#### 生成箇所(where its instances are created)
それぞれ, 以下の箇所でインスタンスが生成されている.

* DefNewGeneration 及びそのサブクラス (ParNewGeneration, ASParNewGeneration)

```
    ((cite: hotspot/src/share/vm/memory/defNewGeneration.cpp))
    DefNewGeneration::DefNewGeneration(ReservedSpace rs,
                                       size_t initial_size,
                                       int level,
                                       const char* policy)
    ...
    {
    ...
      _gc_counters = new CollectorCounters(policy, 0);
```

* TenuredGeneration

```
    ((cite: hotspot/src/share/vm/memory/tenuredGeneration.cpp))
    TenuredGeneration::TenuredGeneration(ReservedSpace rs,
                                         size_t initial_byte_size, int level,
                                         GenRemSet* remset) :
    ...
    {
    ...
      _gc_counters = new CollectorCounters("MSC", 1);
```

* CMSCollector

```
    ((cite: hotspot/src/share/vm/gc_implementation/concurrentMarkSweep/concurrentMarkSweepGeneration.cpp))
    CMSCollector::CMSCollector(ConcurrentMarkSweepGeneration* cmsGen,
                               ConcurrentMarkSweepGeneration* permGen,
                               CardTableRS*                   ct,
                               ConcurrentMarkSweepPolicy*     cp):
    ...
    {
    ...
      _gc_counters = new CollectorCounters("CMS", 1);
```

* G1MonitoringSupport

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1MonitoringSupport.cpp))
    G1MonitoringSupport::G1MonitoringSupport(G1CollectedHeap* g1h,
                                             VirtualSpace* g1_storage_addr) :
    ...
    {
      // Counters for GC collections
      //
      //  name "collector.0".  In a generational collector this would be the
      // young generation collection.
      _incremental_collection_counters =
        new CollectorCounters("G1 incremental collections", 0);
      //   name "collector.1".  In a generational collector this would be the
      // old generation collection.
      _full_collection_counters =
        new CollectorCounters("G1 stop-the-world full collections", 1);
```

* PSScavenge

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    void PSScavenge::initialize() {
    ...
      _counters = new CollectorCounters("PSScavenge", 0);
```

* PSMarkSweep

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psMarkSweep.cpp))
    void PSMarkSweep::initialize() {
    ...
      _counters = new CollectorCounters("PSMarkSweep", 1);
```

* PSParallelCompact

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp))
    void PSParallelCompact::post_initialize() {
    ...
      _counters = new CollectorCounters("PSParallelCompact", 1);
```


### 内部構造(Internal structure)
実体としては内部にいくつかの PerfCounter や PerfVariable を納めたクラス. それらへのアクセサを提供しているだけ.

内部には以下の Perf データを格納している.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp))
        PerfCounter*      _invocations;
        PerfCounter*      _time;
        PerfVariable*     _last_entry_time;
        PerfVariable*     _last_exit_time;
    
        // Constant PerfData types don't need to retain a reference.
        // However, it's a good idea to document them here.
        // PerfStringConstant*     _name;
```

提供しているメソッドは, 内部の Perf データへのアクセサのみ.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp))
        inline PerfCounter* invocation_counter() const  { return _invocations; }
    
        inline PerfCounter* time_counter() const        { return _time; }
    
        inline PerfVariable* last_entry_counter() const { return _last_entry_time; }
    
        inline PerfVariable* last_exit_counter() const  { return _last_exit_time; }
```

また, Perf で公開されたデータには, それぞれ以下の名前でアクセス可能.
(${n} の箇所には 0, 1, ... といった数字が入る. 例えば sun.gc.collector.0.name 等)

  * sun.gc.collector.${n}.name
  * sun.gc.collector.${n}.invocations
  * sun.gc.collector.${n}.time
  * sun.gc.collector.${n}.lastEntryTime
  * sun.gc.collector.${n}.lastExitTime

#### 参考(for your information): コンストラクタの処理
(なお, 見ての通り, このクラスは UsePerfData オプションが指定されている場合にしか動作しない).

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.cpp))
    CollectorCounters::CollectorCounters(const char* name, int ordinal) {
    
      if (UsePerfData) {
        EXCEPTION_MARK;
        ResourceMark rm;
    
        const char* cns = PerfDataManager::name_space("collector", ordinal);
    
        _name_space = NEW_C_HEAP_ARRAY(char, strlen(cns)+1);
        strcpy(_name_space, cns);
    
        char* cname = PerfDataManager::counter_name(_name_space, "name");
        PerfDataManager::create_string_constant(SUN_GC, cname, name, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "invocations");
        _invocations = PerfDataManager::create_counter(SUN_GC, cname,
                                                       PerfData::U_Events, CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "time");
        _time = PerfDataManager::create_counter(SUN_GC, cname, PerfData::U_Ticks,
                                                CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "lastEntryTime");
        _last_entry_time = PerfDataManager::create_variable(SUN_GC, cname,
                                                            PerfData::U_Ticks,
                                                            CHECK);
    
        cname = PerfDataManager::counter_name(_name_space, "lastExitTime");
        _last_exit_time = PerfDataManager::create_variable(SUN_GC, cname,
                                                           PerfData::U_Ticks,
                                                           CHECK);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classCollectorCounters.html) for details

---
## <a name="noANAamRFG" id="noANAamRFG">TraceCollectorStats</a>

### 概要(Summary)
CollectorCounters の記録処理を簡単に行うための補助クラス(StackObjクラス).

CollectorCounters の記録処理をソースコード上のスコープに合わせて自動で行うことができる.


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp))
    class TraceCollectorStats: public PerfTraceTimedEvent {
```

### 使われ方(Usage)
コード中で TraceCollectorStats 型の局所変数を宣言するだけ.

なお, コンストラクタ引数として記録対象の CollectorCounters オブジェクトを受け取る.

### 内部構造(Internal structure)
コンストラクタ/デストラクタ内の処理は以下の通り. CollectorCounters の各カウンタに対して, 以下のように記録処理が行われる
(なお, 見ての通り, このクラスは UsePerfData オプションが指定されている場合にしか動作しない).

* invocation_counter : PerfTraceTimedEvent のコンストラクタによってインクリメントされる
* time_counter : PerfTraceTimedEvent のコンストラクタ/デストラクタによって, 開始時点と終了時点の時刻差が足し込まれる
* last_entry_counter : コンストラクタによって開始時点の時刻が記録される
* last_exit_counter : デストラクタによって終了時点の時刻が記録される

(なお, 時刻の取得には os::elapsed_counter() が用いられている).


```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/collectorCounters.hpp))
        inline TraceCollectorStats(CollectorCounters* c) :
               PerfTraceTimedEvent(c->time_counter(), c->invocation_counter()),
               _c(c) {
    
          if (UsePerfData) {
             _c->last_entry_counter()->set_value(os::elapsed_counter());
          }
        }
    
        inline ~TraceCollectorStats() {
          if (UsePerfData) _c->last_exit_counter()->set_value(os::elapsed_counter());
        }
```




### 詳細(Details)
See: [here](../doxygen/classTraceCollectorStats.html) for details

---
