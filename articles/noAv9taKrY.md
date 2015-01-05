---
layout: default
title: PerfMemory クラス 
---
[Top](../index.html)

#### PerfMemory クラス 



---
## <a name="noe6zgaIox" id="noe6zgaIox">PerfMemory</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス. UsePerfData オプションが指定されている場合にのみ使用される)
(See: [here](no3420acA.html) for details).

PerfData 用の shared memory file ("hsperfdata" ファイル) を管理する機能を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/perfMemory.hpp))
    /* the PerfMemory class manages creation, destruction,
     * and allocation of the PerfData region.
     */
    class PerfMemory : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

<div class="flow-abst"><pre>
* hsperfdata ファイルを作成する処理
  
  (HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
  -&gt; Threads::create_vm()
     -&gt; vm_init_globals()
        -&gt; perfMemory_init()
           -&gt; PerfMemory::initialize()

* hsperfdata ファイルの初期化完了を宣言する処理

  (PerfDataPrologue::accessible フィールドを true に変更する処理. 
   sun.jvmstat.perfdata.monitor.v2_0.isAccessible() が true を返すようになる.)

  (HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
  -&gt; Threads::create_vm()
     -&gt; TraceVmCreationTime::end()
        -&gt; Management::record_vm_startup_time()
           -&gt; PerfMemory::set_accessible()

* PerfData を生成する処理
  
  PerfLong::PerfLong()
  -&gt; PerfData::create_entry()
     -&gt; PerfMemory::alloc()
     -&gt; PerfMemory::mark_updated()

  PerfByteArray::PerfByteArray()
  -&gt; PerfData::create_entry()
     -&gt; (同上)

* hsperfdata ファイルを破棄する処理
  
  (略) (See: <a href="no3420acA.html">here</a> for details)
  -&gt; perfMemory_exit()
     -&gt; PerfMemory::destroy()

* 他 HotSpot の hsperfdata ファイルにアタッチする処理
  
  sun.misc.Perf.attach()
  -&gt; Perf_Attach()
     -&gt; PerfMemory::attach()

* 他 HotSpot の hsperfdata ファイルからデタッチする処理
  
  sun.misc.Perf.detach()
  -&gt; Perf_Detach()
     -&gt; PerfMemory::detach()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classPerfMemory.html) for details

---
