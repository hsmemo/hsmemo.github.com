---
layout: default
title: RuntimeService クラス 
---
[Top](../index.html)

#### RuntimeService クラス 



---
## <a name="no86CXVUcr" id="no86CXVUcr">RuntimeService</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: sun.management.HotspotRuntime).
(See: [here](no2114twV.html) and [here](no2114byp.html) for details)

HotSpot の Runtime の処理(特に safepoint 処理)に関する PerfData を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/services/runtimeService.hpp))
    class RuntimeService : public AllStatic {
```

(なお, sun.management.HotspotRuntime クラスは
 sun.management.HotspotRuntimeMXBean インターフェースの実装に当たるが,
 そもそも sun.management.HotspotRuntimeMXBean 自体が内部的なものであり予告なくインターフェースが変更されうる,
 とのこと.)

```java
    ((cite: jdk/src/share/classes/sun/management/HotspotRuntimeMBean.java))
    /**
     * Hotspot internal management interface for the runtime system.
     *
     * This management interface is internal and uncommitted
     * and subject to change without notice.
     */
    public interface HotspotRuntimeMBean {
```

### 使われ方(Usage)
sun.management.HotspotRuntime クラスから(のみ)使用されている.

(<= ただし内部に持っているのは PerfData なので jcmd 等から見ようと思えば見ることはできるが...)

### 内部構造(Internal structure)
内部には以下のような Perf データを格納している
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).

  * safepoint になった回数 (_total_safepoints)
  * safepoint で費やされた時間 (_safepoint_time_ticks)
  * safepoint になるための同期処理で費やされた時間 (_sync_time_ticks)
  * (safepoint 処理ではなく) 実際の Java アプリケーションの実行に使われた時間 (_application_time_ticks)
  * 
  * 
  * 

```cpp
    ((cite: hotspot/src/share/vm/services/runtimeService.hpp))
      static PerfCounter* _sync_time_ticks;        // Accumulated time spent getting to safepoints
      static PerfCounter* _total_safepoints;
      static PerfCounter* _safepoint_time_ticks;   // Accumulated time at safepoints
      static PerfCounter* _application_time_ticks; // Accumulated time not at safepoints
      static PerfCounter* _thread_interrupt_signaled_count;// os:interrupt thr_kill
      static PerfCounter* _interrupted_before_count;  // _INTERRUPTIBLE OS_INTRPT
      static PerfCounter* _interrupted_during_count;  // _INTERRUPTIBLE OS_INTRPT
```

これらの Perf データには, それぞれ以下の名前でアクセス可能.

  * sun.rt.safepoints
  * sun.rt.safepointTime
  * sun.rt.safepointSyncTime
  * sun.rt.applicationTime
  * sun.rt.threadInterruptSignaled
  * sun.rt.interruptedBeforeIO
  * sun.rt.interruptedDuringIO

```cpp
    ((cite: hotspot/src/share/vm/services/runtimeService.cpp))
        _sync_time_ticks =
                  PerfDataManager::create_counter(SUN_RT, "safepointSyncTime",
                                                  PerfData::U_Ticks, CHECK);
    
        _total_safepoints =
                  PerfDataManager::create_counter(SUN_RT, "safepoints",
                                                  PerfData::U_Events, CHECK);
    
        _safepoint_time_ticks =
                  PerfDataManager::create_counter(SUN_RT, "safepointTime",
                                                  PerfData::U_Ticks, CHECK);
    
        _application_time_ticks =
                  PerfDataManager::create_counter(SUN_RT, "applicationTime",
                                                  PerfData::U_Ticks, CHECK);
    
    
        // create performance counters for jvm_version and its capabilities
        PerfDataManager::create_constant(SUN_RT, "jvmVersion", PerfData::U_None,
                                         (jlong) Abstract_VM_Version::jvm_version(), CHECK);
    
        // I/O interruption related counters
    
        // thread signaling via os::interrupt()
    
        _thread_interrupt_signaled_count =
                    PerfDataManager::create_counter(SUN_RT,
                     "threadInterruptSignaled", PerfData::U_Events, CHECK);
    
        // OS_INTRPT via "check before" in _INTERRUPTIBLE
    
        _interrupted_before_count =
                    PerfDataManager::create_counter(SUN_RT, "interruptedBeforeIO",
                                                    PerfData::U_Events, CHECK);
    
        // OS_INTRPT via "check during" in _INTERRUPTIBLE
    
        _interrupted_during_count =
                    PerfDataManager::create_counter(SUN_RT, "interruptedDuringIO",
                                                    PerfData::U_Events, CHECK);
```

### 備考(Notes)
なお, 以下の PerfData は, Java のクラスからは使われていない.

  * sun.rt.applicationTime
  * sun.rt.threadInterruptSignaled
  * sun.rt.interruptedBeforeIO
  * sun.rt.interruptedDuringIO

sun.rt.applicationTime についてはアクセス用の jmm の関数までは用意されている
(しかしこれらは Java のクラスからは使われていない...).
(他の PerfData についてはアクセス用の jmm 関数も用意されていない).

<div class="flow-abst"><pre>
  -&gt; jmm_GetLongAttribute()
     -&gt; get_long_attribute()  (引数が JMM_TOTAL_APP_TIME_MS の場合)
        -&gt; RuntimeService::application_time_ms()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classRuntimeService.html) for details

---
