---
layout: default
title: ThreadLocalAllocBuffer クラス関連のクラス (ThreadLocalAllocBuffer, GlobalTLABStats)
---
[Top](../index.html)

#### ThreadLocalAllocBuffer クラス関連のクラス (ThreadLocalAllocBuffer, GlobalTLABStats)

これらは, Java ヒープ中でのメモリ確保処理 (= Java オブジェクトの生成処理) を高速化するためのクラス
(See: [here](no28916Q0G.html) and [here](no28916dR0.html) for details).


### クラス一覧(class list)

  * [ThreadLocalAllocBuffer](#noJZ6LtVAE)
  * [GlobalTLABStats](#noz3SsV3MU)


---
## <a name="noJZ6LtVAE" id="noJZ6LtVAE">ThreadLocalAllocBuffer</a>

### 概要(Summary)
Java ヒープ中でのメモリ確保処理 (= Java オブジェクトの生成処理) を高速化するためのクラス.

より具体的に言うと "TLAB" (Thread Local Allocation Buffer) を表すクラス.
1つの ThreadLocalAllocBuffer オブジェクトが 1つの TLAB に対応する.

なお "TLAB" (Thread Local Allocation Buffer) とは, 
各スレッドがスレッドローカルに確保している空きメモリ領域.
メモリ確保の度に毎回排他して大域の空き領域から取得するとオーバーヘッドが大きいので, 
ある程度の大きさの空き領域をスレッドローカルに確保して高速化している.

オブジェクトの生成処理時には各スレッドは自分の TLAB からメモリを切り出す.
これにより TLAB 内に空き領域が残っている間は排他処理が不要になり確保が高速化される.

TLAB が空になったら, 次の TLAB 用の領域を大域の空き領域から取ってくる.
この時には (当然ながら) 排他を取って領域を確保する.


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
    // ThreadLocalAllocBuffer: a descriptor for thread-local storage used by
    // the threads for allocation.
    //            It is thread-private at any time, but maybe multiplexed over
    //            time across multiple threads. The park()/unpark() pair is
    //            used to make it avaiable for such multiplexing.
    class ThreadLocalAllocBuffer: public CHeapObj {
```

### 使われ方(Usage)
#### このクラスの初期化箇所(where this class is initialized)
クラスの初期化は ThreadLocalAllocBuffer::startup_initialization() 内で行われている.

そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; universe_init()
         -&gt; Universe::initialize_heap()
             -&gt; ThreadLocalAllocBuffer::startup_initialization()
</pre></div>

#### インスタンスの格納場所(where its instances are stored)
各 Thread オブジェクトの _tlab フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(Thread クラスの _tlab フィールドは, ポインタ型ではなく実体なので,
 Thread オブジェクトの生成時に一緒に生成される)

#### 初期化箇所(where its instances are initialized)
初期化は ThreadLocalAllocBuffer::initialize() 内で行われている.

そして, この関数は現在は以下のパスで呼び出されている. (#TODO 他のパス)

<div class="flow-abst"><pre>
* HotSpot の起動時処理時 (= メインスレッド用の ThreadLocalAllocBuffer の初期化処理)
  
  (HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
  -&gt; Threads::create_vm()
     -&gt; init_globals()
        -&gt; universe_init()
           -&gt; Universe::initialize_heap()
              -&gt; ThreadLocalAllocBuffer::startup_initialization()
                 -&gt; ThreadLocalAllocBuffer::initialize()

* JavaThread の実行開始時

  JavaThread::run()
  -&gt; Thread::initialize_tlab()
     -&gt; ThreadLocalAllocBuffer::initialize()
</pre></div>

#### 使用箇所(where its instances are used)
以下の箇所で使用されている. (#TODO 他の使用箇所)

<div class="flow-abst"><pre>
* TLAB 内からのオブジェクトの確保処理
  
  (略) (See: <a href="no28916Rgx.html">here</a> for details)
  -&gt; TemplateTable::_new() が生成するコード

  (略) ...(#TODO)
  -&gt; BytecodeInterpreter::run()
     -&gt; ThreadLocalAllocBuffer::allocate()

  (略) ...(#TODO)
  -&gt; ThreadLocalAllocBuffer::allocate()
  
  (略) ...(#TODO)
  
* 新しい TLAB 用領域の確保処理
  
  (略) (See: <a href="no28916Q0G.html">here</a> for details)
  -&gt; CollectedHeap::allocate_from_tlab_slow()
     -&gt; CollectedHeap::allocate_new_tlab() (を各サブクラスがオーバーライドしたもの)
        -&gt; (略) (See: <a href="no28916rXC.html">here</a>, <a href="no289164hI.html">here</a> and <a href="no28916FsO.html">here</a> for details)

* TLAB の動的サイズ調整処理

  (略) (See: <a href="no28916dR0.html">here</a> for details)
  -&gt; ...(#TODO)
</pre></div>

### 内部構造(Internal structure)
内部的には, 以下の4つのアドレスで TLAB として使用するメモリ領域を管理している.

(_start から _end までが TLAB の領域. _top までは使用済 (= _top 以降は未使用))


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
      HeapWord* _start;                              // address of TLAB
      HeapWord* _top;                                // address after last allocation
      HeapWord* _pf_top;                             // allocation prefetch watermark
      HeapWord* _end;                                // allocation end (excluding alignment_reserve)
```

### 備考(Notes)
なお, このクラスは product オプションである UseTLAB が true の場合にしか使用されない. ただしデフォルトでは true.





### 詳細(Details)
See: [here](../doxygen/classThreadLocalAllocBuffer.html) for details

---
## <a name="noz3SsV3MU" id="noz3SsV3MU">GlobalTLABStats</a>

### 概要(Summary)
ThreadLocalAllocBuffer クラス内で使用される補助クラス.

TLAB に関するプロファイル情報を溜めておくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
    class GlobalTLABStats: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
static フィールドである ThreadLocalAllocBuffer::_global_stats に格納されている
(ThreadLocalAllocBuffer::global_stats() がアクセサ).


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
      static GlobalTLABStats* _global_stats;
      static GlobalTLABStats* global_stats() { return _global_stats; }
```

#### 使用箇所(where its instances are used)
内部に格納された情報は, 以下のメソッドで取得／出力することができる.

  * GlobalTLABStats::publish() : 内部の AdaptiveWeightedAverage や PerfVariable に情報を出力する関数. (なお, PerfVariable に出力されるのは UsePerfData オプションがセットされている場合にのみ)
  * GlobalTLABStats::print() : gclog_or_tty 経由で外部に出力する関数. (なお, この関数は PrintTLAB オプションがセットされている場合にのみ呼び出される)
  * 
  * 


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
      // Write all perf counters to the perf_counters
      void publish();
    
      void print();
    
      // Accessors
      unsigned allocating_threads_avg() {
        return MAX2((unsigned)(_allocating_threads_avg.average() + 0.5), 1U);
      }
    
      size_t allocation() {
        return _total_allocation;
      }
```

そして, これらのメソッドは以下の箇所で使用されている.

<div class="flow-abst"><pre>
* TLAB の初期サイズの計算処理

  (See: <a href="no7882jgS.html">here</a> for details)
  -&gt; ThreadLocalAllocBuffer::initial_desired_size()
     -&gt; GlobalTLABStats::allocating_threads_avg()

* 情報の出力処理が行われている

  (See: <a href="no28916dR0.html">here</a> for details)
  -&gt; ThreadLocalAllocBuffer::accumulate_statistics_before_gc()
     -&gt; GlobalTLABStats::initialize()
     -&gt; GlobalTLABStats::allocation()
     -&gt; GlobalTLABStats::publish()
     -&gt; GlobalTLABStats::print()
     -&gt; ThreadLocalAllocBuffer::accumulate_statistics()
        -&gt; GlobalTLABStats::update_allocating_threads()
        -&gt; GlobalTLABStats::update_number_of_refills() 
        -&gt; GlobalTLABStats::update_allocation()	   
        -&gt; GlobalTLABStats::update_gc_waste()	   
        -&gt; GlobalTLABStats::update_slow_refill_waste() 
        -&gt; GlobalTLABStats::update_slow_allocations()  
</pre></div>

### 内部構造(Internal structure)
内部では, (UsePerfData オプションが指定されている場合には) PerfVariable も使用している.


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.hpp))
      PerfVariable* _perf_allocating_threads;
      PerfVariable* _perf_total_refills;
      PerfVariable* _perf_max_refills;
      PerfVariable* _perf_allocation;
      PerfVariable* _perf_gc_waste;
      PerfVariable* _perf_max_gc_waste;
      PerfVariable* _perf_slow_refill_waste;
      PerfVariable* _perf_max_slow_refill_waste;
      PerfVariable* _perf_fast_refill_waste;
      PerfVariable* _perf_max_fast_refill_waste;
      PerfVariable* _perf_slow_allocations;
      PerfVariable* _perf_max_slow_allocations;
```

これらの PerfVariable には, それぞれ以下の名前でアクセス可能.

* sun.gc.tlab.allocThreads
* sun.gc.tlab.fills
* sun.gc.tlab.maxFills
* sun.gc.tlab.alloc
* sun.gc.tlab.gcWaste
* sun.gc.tlab.maxGcWaste
* sun.gc.tlab.slowWaste
* sun.gc.tlab.maxSlowWaste
* sun.gc.tlab.fastWaste
* sun.gc.tlab.maxFastWaste
* sun.gc.tlab.slowAlloc
* sun.gc.tlab.maxSlowAlloc


```cpp
    ((cite: hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp))
        char* cname = PerfDataManager::counter_name("tlab", "allocThreads");
        _perf_allocating_threads =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "fills");
        _perf_total_refills =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "maxFills");
        _perf_max_refills =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "alloc");
        _perf_allocation =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "gcWaste");
        _perf_gc_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "maxGcWaste");
        _perf_max_gc_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "slowWaste");
        _perf_slow_refill_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "maxSlowWaste");
        _perf_max_slow_refill_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "fastWaste");
        _perf_fast_refill_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "maxFastWaste");
        _perf_max_fast_refill_waste =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_Bytes, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "slowAlloc");
        _perf_slow_allocations =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None, CHECK);
    
        cname = PerfDataManager::counter_name("tlab", "maxSlowAlloc");
        _perf_max_slow_allocations =
          PerfDataManager::create_variable(SUN_GC, cname, PerfData::U_None, CHECK);
      }
```




### 詳細(Details)
See: [here](../doxygen/classGlobalTLABStats.html) for details

---
