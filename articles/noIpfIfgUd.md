---
layout: default
title: ConcurrentG1Refine クラス 
---
[Top](../index.html)

#### ConcurrentG1Refine クラス 



---
## <a name="noS-BL9lTx" id="noS-BL9lTx">ConcurrentG1Refine</a>

### 概要(Summary)
G1CollectedHeap クラス内で使用される補助クラス.

ConcurrentG1RefineThread オブジェクトの管理を行う
(ConcurrentG1RefineThread オブジェクトは一般に複数存在するが, それらをこのクラスでまとめて管理する).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.hpp))
    class ConcurrentG1Refine: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _cg1r フィールドに(のみ)格納されている
(「各」と言っても1つしかいないが...)
(アクセサは G1CollectedHeap::concurrent_g1_refine()).

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.

### 内部構造(Internal structure)
ConcurrentG1RefineThread オブジェクトは以下の配列に格納されている
(その下にあるのは配列の要素数を表すフィールド).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.hpp))
      ConcurrentG1RefineThread** _threads;
      int _n_threads;
      int _n_worker_threads;
```

また, filled RS buffers がどこまでたまったら 
ConcurrentG1RefineThread (や Mutator 自身の refine 処理) を起動させるかは,
以下のフィールドの値で決まる.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.hpp))
     /*
      * The value of the update buffer queue length falls into one of 3 zones:
      * green, yellow, red. If the value is in [0, green) nothing is
      * done, the buffers are left unprocessed to enable the caching effect of the
      * dirtied cards. In the yellow zone [green, yellow) the concurrent refinement
      * threads are gradually activated. In [yellow, red) all threads are
      * running. If the length becomes red (max queue length) the mutators start
      * processing the buffers.
      *
      * There are some interesting cases (when G1UseAdaptiveConcRefinement
      * is turned off):
      * 1) green = yellow = red = 0. In this case the mutator will process all
      *    buffers. Except for those that are created by the deferred updates
      *    machinery during a collection.
      * 2) green = 0. Means no caching. Can be a good way to minimize the
      *    amount of time spent updating rsets during a collection.
      */
      int _green_zone;
      int _yellow_zone;
      int _red_zone;
```

その他に, ConcurrentG1RefineThread を停止させる stop() メソッドや
iterate するための threads_do() メソッド等を備えている.

また, hot card を覚えておくためのキャッシュの管理も行っている
(ConcurrentG1Refine::use_cache(), ConcurrentG1Refine::cache_insert(), etc).




### 詳細(Details)
See: [here](../doxygen/classConcurrentG1Refine.html) for details

---
