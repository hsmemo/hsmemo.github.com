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


```
    ((cite: hotspot/src/share/vm/runtime/perfMemory.hpp))
    /* the PerfMemory class manages creation, destruction,
     * and allocation of the PerfData region.
     */
    class PerfMemory : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* hsperfdata ファイルを作成する処理
  
  (HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
  -> Threads::create_vm()
     -> vm_init_globals()
        -> perfMemory_init()
           -> PerfMemory::initialize()

* hsperfdata ファイルの初期化完了を宣言する処理

  (PerfDataPrologue::accessible フィールドを true に変更する処理. 
   sun.jvmstat.perfdata.monitor.v2_0.isAccessible() が true を返すようになる.)

  (HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
  -> Threads::create_vm()
     -> TraceVmCreationTime::end()
        -> Management::record_vm_startup_time()
           -> PerfMemory::set_accessible()

* PerfData を生成する処理
  
  PerfLong::PerfLong()
  -> PerfData::create_entry()
     -> PerfMemory::alloc()
     -> PerfMemory::mark_updated()

  PerfByteArray::PerfByteArray()
  -> PerfData::create_entry()
     -> (同上)

* hsperfdata ファイルを破棄する処理
  
  (略) (See: [here](no3420acA.html) for details)
  -> perfMemory_exit()
     -> PerfMemory::destroy()

* 他 HotSpot の hsperfdata ファイルにアタッチする処理
  
  sun.misc.Perf.attach()
  -> Perf_Attach()
     -> PerfMemory::attach()

* 他 HotSpot の hsperfdata ファイルからデタッチする処理
  
  sun.misc.Perf.detach()
  -> Perf_Detach()
     -> PerfMemory::detach()
```




### 詳細(Details)
See: [here](../doxygen/classPerfMemory.html) for details

---
