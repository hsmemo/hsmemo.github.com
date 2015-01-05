---
layout: default
title: StatSampler クラス (StatSampler, 及びその補助クラス(StatSamplerTask, HighResTimeSampler))
---
[Top](../index.html)

#### StatSampler クラス (StatSampler, 及びその補助クラス(StatSamplerTask, HighResTimeSampler))



### クラス一覧(class list)

  * [StatSampler](#nol5pI-Ojb)
  * [StatSamplerTask](#noHHwXqyFi)
  * [HighResTimeSampler](#noAio-DJef)


---
## <a name="nol5pI-Ojb" id="nol5pI-Ojb">StatSampler</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData 管理用のクラス)
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

定期的に PerfData のデータを共有メモリ(hsperfdataファイル)に反映させるためのメソッドを格納している
(See: [here](no3420acA.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/statSampler.hpp))
    /*
     * The StatSampler class is responsible for periodically updating
     * sampled PerfData instances and writing the sampled values to the
     * PerfData memory region.
     *
     * In addition it is also responsible for providing a home for
     * PerfData instances that otherwise have no better home.
     */
    class StatSampler : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* HotSpot の起動時

  Threads::create_vm() の内で StatSampler::engage() が呼ばれて処理が開始される.


```cpp
    ((cite: hotspot/src/share/vm/runtime/thread.cpp))
    jint Threads::create_vm(JavaVMInitArgs* args, bool* canTryAgain) {
    ...
      StatSampler::engage();
```

* HotSpot の実行中

  StatSamplerTask::task() で, 定期間隔で StatSampler::collect_sample() が呼ばれてファイルへの出力が行われる.

* HotSpot の終了時

  before_exit() の内で StatSampler::disengage() 及び StatSampler::destroy() が呼ばれて後片付けが行われる.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void before_exit(JavaThread * thread) {
    ...
      // shut down the StatSampler task
      StatSampler::disengage();
      StatSampler::destroy();
```




### 詳細(Details)
See: [here](../doxygen/classStatSampler.html) for details

---
## <a name="noHHwXqyFi" id="noHHwXqyFi">StatSamplerTask</a>

### 概要(Summary)
StatSampler クラス内で使用される補助クラス.

定期間隔で PerfData のデータを共有メモリ(hsperfdataファイル)に反映させるためのクラス(PeriodicTaskクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/statSampler.cpp))
    // --------------------------------------------------------
    // StatSamplerTask
    
    class StatSamplerTask : public PeriodicTask {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
StatSampler クラスの _task フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
StatSampler::engage() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定期間隔で StatSampler::collect_sample() を呼び出しているだけ.




### 詳細(Details)
See: [here](../doxygen/classStatSamplerTask.html) for details

---
## <a name="noAio-DJef" id="noAio-DJef">HighResTimeSampler</a>

### 概要(Summary)
保守運用機能のためのクラス (PerfData による情報出力用のクラス).

os クラスが管理している経過時間情報 (os::elapsed_counter()) を 
PerfData から出力するための補助クラス(PerfSampleHelperクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/statSampler.cpp))
    /*
     * helper class to provide for sampling of the elapsed_counter value
     * maintained in the OS class.
     */
    class HighResTimeSampler : public PerfSampleHelper {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
StatSampler::create_sampled_perfdata() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; StatSampler::engage()
      -&gt; StatSampler::initialize()
         -&gt; StatSampler::create_misc_perfdata()
            -&gt; StatSampler::create_sampled_perfdata()
</pre></div>

#### 使用箇所(where its instances are used)
"sun.os.hrt.ticks" という名前の PerfCounter を作る際に, コンストラクタ引数として渡される
(これにより定期間隔で HighResTimeSampler::take_sample() が呼び出される)
(See: StatSampler::create_sampled_perfdata()).

### 内部構造(Internal structure)
os::elapsed_counter() を呼び出すだけ.




### 詳細(Details)
See: [here](../doxygen/classHighResTimeSampler.html) for details

---
