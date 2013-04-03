---
layout: default
title: LowMemoryDetector クラス関連のクラス (ThresholdSupport, SensorInfo, LowMemoryDetector, LowMemoryDetectorDisabler)
---
[Top](../index.html)

#### LowMemoryDetector クラス関連のクラス (ThresholdSupport, SensorInfo, LowMemoryDetector, LowMemoryDetectorDisabler)

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと,
java.lang.management.MemoryPoolMXBean.setUsageThreshold() メソッド, 及び
java.lang.management.MemoryPoolMXBean.setCollectionUsageThreshold() メソッドの実装を担当するクラス.
(See: [here](no2114twV.html) for details)
(See: [here](no2114x0x.html) for details)

### 備考(Notes)
java.lang.management.MemoryPoolMXBean.setUsageThreshold() メソッド, 及び
java.lang.management.MemoryPoolMXBean.setCollectionUsageThreshold() メソッドで指定する threshold 値は 
0 より大きな値でなければならない.
これを利用して, 特殊な内部状態を以下のようにエンコードしている.

* threshold の値が -1 の状態は, 閾値超過機能に対応していない MemoryPool であることを示す.
* threshold の値が 0 の状態は, (閾値超過機能に対応しているが) 閾値が設定されていない状態を示す.

なお, それぞれの MemoryPool に対する threshold のデフォルト値は以下の通り.

MemoryPool            | threshold
--------------------- | -----------------------------------------------------------------
Eden space       	  | -1
Survivor space 1 	  | -1
Survivor space 2 	  | -1
Old generation   	  |  0
Perm generation  	  |  0
CodeCache        	  |  0

閾値を超過したかどうかのチェックは以下のタイミングで行われる.

* Java ヒープ: GC 終了時, およびメモリ確保処理の slow-path 内
* CodeCache : メモリの確保時と解放時

なお, この機能は ServiceThread と連携して動作する (See: ServiceThread).

```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
    // Low Memory Detection Support
    // Two memory alarms in the JDK (we called them sensors).
    //   - Heap memory sensor
    //   - Non-heap memory sensor
    // When the VM detects if the memory usage of a memory pool has reached
    // or exceeded its threshold, it will trigger the sensor for the type
    // of the memory pool (heap or nonheap or both).
    //
    // If threshold == -1, no low memory detection is supported and
    // the threshold value is not allowed to be changed.
    // If threshold == 0, no low memory detection is performed for
    // that memory pool.  The threshold can be set to any non-negative
    // value.
    //
    // The default threshold of the Hotspot memory pools are:
    //   Eden space        -1
    //   Survivor space 1  -1
    //   Survivor space 2  -1
    //   Old generation    0
    //   Perm generation   0
    //   CodeCache         0
    //
    // For heap memory, detection will be performed when GC finishes
    // and also in the slow path allocation.
    // For Code cache, detection will be performed in the allocation
    // and deallocation.
    //
    // May need to deal with hysteresis effect.
    //
    // Memory detection code runs in the Service thread (serviceThread.hpp).
```


### クラス一覧(class list)

  * [LowMemoryDetector](#nollsiFS4C)
  * [ThresholdSupport](#noQhvSdnjK)
  * [SensorInfo](#noE3kmdQEE)
  * [LowMemoryDetectorDisabler](#noZnriBYPu)


---
## <a name="nollsiFS4C" id="nollsiFS4C">LowMemoryDetector</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: java.lang.management.MemoryPoolMXBean)
(See: [here](no2114x0x.html) for details)

メモリ使用量に関する閾値超過検出処理用の関数を納めた名前空間(AllStatic クラス).

```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
    class LowMemoryDetector : public AllStatic {
```

### 使われ方(Usage)
java.lang.management.MemoryPoolMXBean.setUsageThreshold() メソッド, 及び
java.lang.management.MemoryPoolMXBean.setCollectionUsageThreshold() メソッドの処理で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classLowMemoryDetector.html) for details

---
## <a name="noQhvSdnjK" id="noQhvSdnjK">ThresholdSupport</a>

### 概要(Summary)
LowMemoryDetector クラス内で使用される補助クラス.

各 MemoryPool オブジェクトの閾値情報を記録しておくためのクラス.

```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
    class ThresholdSupport : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MemoryPool オブジェクトの _usage_threshold フィールドおよび
_gc_usage_threshold フィールドに(のみ)格納されている
(それぞれ, setUsageThreshold() メソッド用と setCollectionUsageThreshold() メソッド用).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class MemoryPool : public CHeapObj {
    ...
      ThresholdSupport* _usage_threshold;
      ThresholdSupport* _gc_usage_threshold;
```




### 詳細(Details)
See: [here](../doxygen/classThresholdSupport.html) for details

---
## <a name="noE3kmdQEE" id="noE3kmdQEE">SensorInfo</a>

### 概要(Summary)
LowMemoryDetector クラス内で使用される補助クラス.

sun.management.Sensor クラスを実現するためのクラス.
Sensor オブジェクトに設定する値を HotSpot 内で記録しておくために使われる.


```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
    class SensorInfo : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MemoryPool オブジェクトの _usage_sensor フィールドおよび
_gc_usage_sensor フィールドに(のみ)格納されている
(それぞれ, setUsageThreshold() メソッド用と setCollectionUsageThreshold() メソッド用).


```
    ((cite: hotspot/src/share/vm/services/memoryPool.hpp))
    class MemoryPool : public CHeapObj {
    ...
      SensorInfo*      _usage_sensor;
      SensorInfo*      _gc_usage_sensor;
```




### 詳細(Details)
See: [here](../doxygen/classSensorInfo.html) for details

---
## <a name="noZnriBYPu" id="noZnriBYPu">LowMemoryDetectorDisabler</a>

### 概要(Summary)
?? (使われていないクラス)

ソースコード中のあるスコープの間だけ, LowMemoryDetector の機能を無効にするためのクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
    class LowMemoryDetectorDisabler: public StackObj {
```

### 使われ方(Usage)
(現状では使われていない)

### 内部構造(Internal structure)
コンストラクタで LowMemoryDetector::disable() を呼び出し, 
デストラクタで LowMemoryDetector::enable() を呼び出すだけ.

```
    ((cite: hotspot/src/share/vm/services/lowMemoryDetector.hpp))
      LowMemoryDetectorDisabler()
      {
        LowMemoryDetector::disable();
      }
      ~LowMemoryDetectorDisabler()
      {
        assert(LowMemoryDetector::temporary_disabled(), "should be disabled!");
        LowMemoryDetector::enable();
      }
```




### 詳細(Details)
See: [here](../doxygen/classLowMemoryDetectorDisabler.html) for details

---
