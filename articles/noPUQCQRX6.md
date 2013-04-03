---
layout: default
title: JMM 関連の雑多なクラス (Management, TraceVmCreationTime, VmThreadCountClosure, ThreadTimesClosure)
---
[Top](../index.html)

#### JMM 関連の雑多なクラス (Management, TraceVmCreationTime, VmThreadCountClosure, ThreadTimesClosure)

これらは, 保守運用機能のためのクラス.
より具体的に言うと, JMM 機能のためのクラス (See: [here](no2114S_x.html) for details).


### クラス一覧(class list)

  * [Management](#no0hTynk2W)
  * [TraceVmCreationTime](#noW9pSrncO)
  * [VmThreadCountClosure](#nogqop6YFe)
  * [ThreadTimesClosure](#nolV3brukv)


---
## <a name="no0hTynk2W" id="no0hTynk2W">Management</a>

### 概要(Summary)
保守運用機能のためのクラス (JMM 機能用のクラス).

JMM に関係する雑多な処理(初期化処理など)や klassOop 等を納めた名前空間(AllStatic クラス) (See: [here](no2114S_x.html) for details).


```
    ((cite: hotspot/src/share/vm/services/management.hpp))
    class Management : public AllStatic {
```

### 内部構造(Internal structure)
内部には, JMM の処理に関連する klassOop への参照を保持している.


```
    ((cite: hotspot/src/share/vm/services/management.hpp))
      // Management klasses
      static klassOop           _sensor_klass;
      static klassOop           _threadInfo_klass;
      static klassOop           _memoryUsage_klass;
      static klassOop           _memoryPoolMXBean_klass;
      static klassOop           _memoryManagerMXBean_klass;
      static klassOop           _garbageCollectorMXBean_klass;
      static klassOop           _managementFactory_klass;
      static klassOop           _garbageCollectorImpl_klass;
      static klassOop           _gcInfo_klass;
```

また, 内部には以下のような Perf データを格納している.


```
    ((cite: hotspot/src/share/vm/services/management.hpp))
      static PerfVariable*      _begin_vm_creation_time;
      static PerfVariable*      _end_vm_creation_time;
      static PerfVariable*      _vm_init_done_time;
```

Perf で公開されたデータには, それぞれ以下の名前でアクセス可能.

* sun.rt.createVmBeginTime
* sun.rt.createVmEndTime
* sun.rt.vmInitDoneTime


```
    ((cite: hotspot/src/share/vm/services/management.cpp))
      _begin_vm_creation_time =
                PerfDataManager::create_variable(SUN_RT, "createVmBeginTime",
                                                 PerfData::U_None, CHECK);
    
      _end_vm_creation_time =
                PerfDataManager::create_variable(SUN_RT, "createVmEndTime",
                                                 PerfData::U_None, CHECK);
    
      _vm_init_done_time =
                PerfDataManager::create_variable(SUN_RT, "vmInitDoneTime",
                                                 PerfData::U_None, CHECK);
```




### 詳細(Details)
See: [here](../doxygen/classManagement.html) for details

---
## <a name="noW9pSrncO" id="noW9pSrncO">TraceVmCreationTime</a>

### 概要(Summary)
Management クラス用の補助クラス. 

HotSpot の起動処理(Threads::create_vm())に掛かった時間を計測するための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/services/management.hpp))
    class TraceVmCreationTime : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
TraceVmCreationTime::start() メソッドで計測が始まる.
TraceVmCreationTime::end() メソッドで終了する.

#### 使用箇所(where its instances are used)
Threads::create_vm() 内で使用されている (See: [here](no2114J7x.html) for details).

### 内部構造(Internal structure)
内部的には, 計測終了時に Management::record_vm_startup_time() を呼び出して
Management::_begin_vm_creation_time フィールド及び
Management::_end_vm_creation_time フィールドの Perf データに記録しているだけ.

#### 参考(for your information): TraceVmCreationTime::start()
See: [here](no31150lpV.html) for details
#### 参考(for your information): TraceVmCreationTime::end()
See: [here](no31150yzb.html) for details
#### 参考(for your information): Management::record_vm_startup_time()
See: [here](no31150_9h.html) for details



### 詳細(Details)
See: [here](../doxygen/classTraceVmCreationTime.html) for details

---
## <a name="nogqop6YFe" id="nogqop6YFe">VmThreadCountClosure</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: sun.management.HotspotThread)
(See: [here](no2114x0x.html) for details)

sun.management.HotspotThreadMBean.getInternalThreadCount() の処理で使用される Closure.
現在存在しているスレッドの合計数を数える (See: [here](no2114vml.html) for details).


```
    ((cite: hotspot/src/share/vm/services/management.cpp))
    class VmThreadCountClosure: public ThreadClosure {
```

### 使われ方(Usage)
get_vm_thread_count() 内で(のみ)使用されている (See: [here](no2114vml.html) for details).

### 内部構造(Internal structure)
VmThreadCountClosure::do_thread() ではカウンタをインクリメントしているだけ
(このメソッドが各スレッドに対して一度ずつ呼び出されるので合計スレッド数が数えられる).




### 詳細(Details)
See: [here](../doxygen/classVmThreadCountClosure.html) for details

---
## <a name="nolV3brukv" id="nolV3brukv">ThreadTimesClosure</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: sun.management.HotspotThread)
(See: [here](no2114x0x.html) for details)

sun.management.HotspotThread.getInternalThreadCpuTimes() の処理で使用される Closure.
各スレッドのスレッド名と CPU 使用時間を取得する (See: [here](no2114vml.html) for details).


```
    ((cite: hotspot/src/share/vm/services/management.cpp))
    class ThreadTimesClosure: public ThreadClosure {
```

### 使われ方(Usage)
jmm_GetInternalThreadTimes() 内で(のみ)使用されている (See: [here](no2114vml.html) for details).




### 詳細(Details)
See: [here](../doxygen/classThreadTimesClosure.html) for details

---
