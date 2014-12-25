---
layout: default
title: CollectorPolicy クラス関連のクラス (CollectorPolicy, ClearedAllSoftRefs, GenCollectorPolicy, TwoGenerationCollectorPolicy, MarkSweepPolicy)
---
[Top](../index.html)

#### CollectorPolicy クラス関連のクラス (CollectorPolicy, ClearedAllSoftRefs, GenCollectorPolicy, TwoGenerationCollectorPolicy, MarkSweepPolicy)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, GC に関する「ポリシー」を表すクラス.
使用する GC アルゴリズムの種別や GC が必要とする情報(Java ヒープの大きさ等)を管理している (See: [here](no3718kvd.html) for details). 
(See: [here](no2114hIm.html), [here](no2114uSs.html), [here](no2114tfN.html) and [here](no2114gVH.html) for details)

なおコメントによると, 
これらのクラスは完成したものではなく, 
新しい GC アルゴリズムによって必要な情報が増えれば今後も拡張されていく, とのこと.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    // This class (or more correctly, subtypes of this class)
    // are used to define global garbage collector attributes.
    // This includes initialization of generations and any other
    // shared resources they may need.
    //
    // In general, all flag adjustment and validation should be
    // done in initialize_flags(), which is called prior to
    // initialize_size_info().
    //
    // This class is not fully developed yet. As more collector(s)
    // are added, it is expected that we will come across further
    // behavior that requires global attention. The correct place
    // to deal with those issues is this class.
```



### クラス一覧(class list)

  * [CollectorPolicy](#no62b4-vIe)
  * [GenCollectorPolicy](#nolR3zDjnn)
  * [TwoGenerationCollectorPolicy](#no1sRpYF5Q)
  * [MarkSweepPolicy](#no190xhxpO)
  * [ClearedAllSoftRefs](#noVI1pIMOg)


---
## <a name="no62b4-vIe" id="no62b4-vIe">CollectorPolicy</a>

### 概要(Summary)
全ての CollectorPolicy クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    class CollectorPolicy : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classCollectorPolicy.html) for details

---
## <a name="nolR3zDjnn" id="nolR3zDjnn">GenCollectorPolicy</a>

### 概要(Summary)
CollectorPolicy クラスのサブクラスの1つ.

このクラスは 世代別 GC (Generational GC) を用いる場合用.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    class GenCollectorPolicy : public CollectorPolicy {
```




### 詳細(Details)
See: [here](../doxygen/classGenCollectorPolicy.html) for details

---
## <a name="no1sRpYF5Q" id="no1sRpYF5Q">TwoGenerationCollectorPolicy</a>

### 概要(Summary)
GenCollectorPolicy クラスのサブクラス (なお, 現在はこのクラスが唯一のサブクラス).

このクラスは 2 世代の 世代別 GC (Generational GC) を用いる場合用.

(なおコメントでは, 全ての CollectorPolicy はこのクラスのサブクラスになっている, と書かれているが, 
G1CollectorPolicy は違うんだが...#TODO)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    // All of hotspot's current collectors are subtypes of this
    // class. Currently, these collectors all use the same gen[0],
    // but have different gen[1] types. If we add another subtype
    // of CollectorPolicy, this class should be broken out into
    // its own file.
    
    class TwoGenerationCollectorPolicy : public GenCollectorPolicy {
```




### 詳細(Details)
See: [here](../doxygen/classTwoGenerationCollectorPolicy.html) for details

---
## <a name="no190xhxpO" id="no190xhxpO">MarkSweepPolicy</a>

### 概要(Summary)
TwoGenerationCollectorPolicy クラスの具象サブクラスの1つ.

このクラスは Serial Old GC を用いる場合用 (= ParallelScavenge でも G1GC でも CMS でもない場合用) (See: [here](no2114hIm.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    class MarkSweepPolicy : public TwoGenerationCollectorPolicy {
```




### 詳細(Details)
See: [here](../doxygen/classMarkSweepPolicy.html) for details

---
## <a name="noVI1pIMOg" id="noVI1pIMOg">ClearedAllSoftRefs</a>

### 概要(Summary)
CollectorPolicy 用の補助クラス.

GC によって soft reference が消去された場合に, 
CollectorPolicy オブジェクトの _all_soft_refs_clear フィールドをセットする処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
    class ClearedAllSoftRefs : public StackObj {
```

なお, CollectorPolicy::_all_soft_refs_clear フィールドは, 
「直近の GC 処理によって全ての soft reference が消去されたかどうか」を示すフィールド (消去されていると true がセットされる).
(See: CollectorPolicy::all_soft_refs_clear())


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
      // Set to true by the GC if the just-completed gc cleared all
      // softrefs.  This is set to true whenever a gc clears all softrefs, and
      // set to false each time gc returns to the mutator.  For example, in the
      // ParallelScavengeHeap case the latter would be done toward the end of
      // mem_allocate() where it returns op.result()
      bool _all_soft_refs_clear;
```

### 使われ方(Usage)
GC 処理中で局所変数として生成される. なお, コンストラクタ引数として以下の2つを受け取る.

* 「soft reference が消去されるかどうか」
* 「消去された場合に _all_soft_refs_clear フィールドを書き換える CollectorPolicy オブジェクト」

生成されたスコープが終わる時点で, デストラクタによってフィールドの値がセットされる.


```cpp
    ((cite: hotspot/src/share/vm/memory/genCollectedHeap.cpp))
    void GenCollectedHeap::do_collection(bool  full,
                                         bool   clear_all_soft_refs,
                                         size_t size,
                                         bool   is_tlab,
                                         int    max_level) {
    ...
      const bool do_clear_all_soft_refs = clear_all_soft_refs ||
                              collector_policy()->should_clear_all_soft_refs();
    
      ClearedAllSoftRefs casr(do_clear_all_soft_refs, collector_policy());
```

### 内部構造(Internal structure)
実際の処理は以下の通り. 
コンストラクタの clear_all_soft_refs 引数が true だった場合には, 
デストラクタで CollectorPolicy::cleared_all_soft_refs() の呼び出しが行われる.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
      ClearedAllSoftRefs(bool clear_all_soft_refs,
                         CollectorPolicy* collector_policy) :
        _clear_all_soft_refs(clear_all_soft_refs),
        _collector_policy(collector_policy) {}
    
      ~ClearedAllSoftRefs() {
        if (_clear_all_soft_refs) {
          _collector_policy->cleared_all_soft_refs();
        }
      }
```

なお, CollectorPolicy::cleared_all_soft_refs() は以下のような関数
(GC 時に soft reference が消去されると, _should_clear_all_soft_refs が満たされたことを示すために呼び出される).


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.hpp))
      // Called by the GC after Soft Refs have been cleared to indicate
      // that the request in _should_clear_all_soft_refs has been fulfilled.
      void cleared_all_soft_refs();
```

実際の CollectorPolicy::cleared_all_soft_refs() の中身は以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/memory/collectorPolicy.cpp))
    void CollectorPolicy::cleared_all_soft_refs() {
      // If near gc overhear limit, continue to clear SoftRefs.  SoftRefs may
      // have been cleared in the last collection but if the gc overhear
      // limit continues to be near, SoftRefs should still be cleared.
      if (size_policy() != NULL) {
        _should_clear_all_soft_refs = size_policy()->gc_overhead_limit_near();
      }
      _all_soft_refs_clear = true;
    }
```




### 詳細(Details)
See: [here](../doxygen/classClearedAllSoftRefs.html) for details

---
