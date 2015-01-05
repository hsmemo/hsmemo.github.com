---
layout: default
title: AllocationProfiler クラス (AllocationProfiler, 及びその補助クラス(AllocProfClosure, AllocProfResetClosure))
---
[Top](../index.html)

#### AllocationProfiler クラス (AllocationProfiler, 及びその補助クラス(AllocProfClosure, AllocProfResetClosure))

これらは, 保守運用機能のためのクラス.
メモリ確保に関する簡単なプロファイラ機能を提供する.


### クラス一覧(class list)

  * [AllocationProfiler](#nog6-caxVM)
  * [AllocProfClosure](#noirQe_j-L)
  * [AllocProfResetClosure](#noApqUA0-_)


---
## <a name="nog6-caxVM" id="nog6-caxVM">AllocationProfiler</a>

### 概要(Summary)
保守運用機能のためのクラス (関連するオプションが指定されている場合にのみ使用される) (See: -Xaprof).

メモリ確保に関する簡単なプロファイラ機能を納めた名前空間(AllStatic クラス).
クラス毎の生成されたインスタンスの合計数およびそれらの合計量(byte)を記録/出力する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/aprofiler.hpp))
    // A simple allocation profiler for Java. The profiler collects and prints
    // the number and total size of instances allocated per class, including
    // array classes.
    //
    // The profiler is currently global for all threads. It can be changed to a
    // per threads profiler by keeping a more elaborate data structure and calling
    // iterate_since_last_scavenge at thread switches.
    
    
    class AllocationProfiler: AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. AllocationProfiler::engage() で, 計測を開始する.
2. AllocationProfiler::iterate_since_last_gc() を呼ぶと, そのたびにデータを収集する.
3. AllocationProfiler::disengage() で, 計測を終了する.
4. AllocationProfiler::print() で, 結果を出力する.
    
#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

<div class="flow-abst"><pre>
* 初期化時に, 計測が開始される.
  
  Threads::create_vm()
  -&gt; AllocationProfiler::engage()

* G1CollectedHeap または GenCollectedHeap 使用時にのみ, GC 前に情報が出力される.
  
  G1CollectedHeap::gc_prologue()
  -&gt; AllocationProfiler::iterate_since_last_gc()
  
  GenCollectedHeap::gc_prologue()
  -&gt; AllocationProfiler::iterate_since_last_gc()

* HotSpot の終了時に, 計測を終了し, 情報が出力される.
  
  before_exit()
  -&gt; AllocationProfiler::disengage()
  -&gt; AllocationProfiler::print()
</pre></div>



### 詳細(Details)
See: [here](../doxygen/classAllocationProfiler.html) for details

---
## <a name="noirQe_j-L" id="noirQe_j-L">AllocProfClosure</a>

### 概要(Summary)
AllocationProfiler クラス内で使用される補助クラス.

クラス毎の生成されたインスタンスの合計数およびそれらの合計量(byte)を各クラスオブジェクト内に記録する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/aprofiler.cpp))
    class AllocProfClosure : public ObjectClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* AllocationProfiler::iterate_since_last_gc()
* AllocationProfiler::print() (ただし, #ifndef PRODUCT ASSERT 時にしか使用されない)




### 詳細(Details)
See: [here](../doxygen/classAllocProfClosure.html) for details

---
## <a name="noApqUA0-_" id="noApqUA0-_">AllocProfResetClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

AllocationProfiler クラス内で使用される補助クラス.

各クラスオブジェクト内の生成されたインスタンスの合計数およびそれらの合計量(byte)情報を 0 にリセットする.


```cpp
    ((cite: hotspot/src/share/vm/runtime/aprofiler.cpp))
    #ifndef PRODUCT
    
    class AllocProfResetClosure : public ObjectClosure {
```

### 使われ方(Usage)
AllocationProfiler::print() 内で(のみ)使用されている
(ただし, #ifndef PRODUCT ASSERT 時にしか使用されない).




### 詳細(Details)
See: [here](../doxygen/classAllocProfResetClosure.html) for details

---
