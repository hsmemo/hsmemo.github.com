---
layout: default
title: ReferenceProcessor クラス関連のクラス (ReferenceProcessor, NoRefDiscovery, ReferenceProcessorSpanMutator, ReferenceProcessorMTDiscoveryMutator, ReferenceProcessorIsAliveMutator, ReferenceProcessorAtomicMutator, ReferenceProcessorMTProcMutator, AbstractRefProcTaskExecutor, AbstractRefProcTaskExecutor::ProcessTask, AbstractRefProcTaskExecutor::EnqueueTask, 及びそれらの補助クラス(DiscoveredList, AlwaysAliveClosure, CountHandleClosure, RefProcEnqueueTask, DiscoveredListIterator, RefProcPhase1Task, RefProcPhase2Task, RefProcPhase3Task))
---
[Top](../index.html)

#### ReferenceProcessor クラス関連のクラス (ReferenceProcessor, NoRefDiscovery, ReferenceProcessorSpanMutator, ReferenceProcessorMTDiscoveryMutator, ReferenceProcessorIsAliveMutator, ReferenceProcessorAtomicMutator, ReferenceProcessorMTProcMutator, AbstractRefProcTaskExecutor, AbstractRefProcTaskExecutor::ProcessTask, AbstractRefProcTaskExecutor::EnqueueTask, 及びそれらの補助クラス(DiscoveredList, AlwaysAliveClosure, CountHandleClosure, RefProcEnqueueTask, DiscoveredListIterator, RefProcPhase1Task, RefProcPhase2Task, RefProcPhase3Task))

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, 参照オブジェクト (java.lang.ref オブジェクト) の GC 処理を行うクラス (See: [here](no289169tf.html) for details).

### 概要(Summary)
ReferenceProcessor クラスは, 参照オブジェクト(java.lang.ref.Reference オブジェクト)の GC 処理を行うクラス.

世代別 GC に対応するため, ヒープの一部分だけを対象とする機能も持っている
(なお内部的には世代別だけに特化しないよう, "世代" ではなく "span" という用語を用いている).

各 ReferenceProcessor オブジェクトは, 指定された GC と対応づけられ, 
指定の "span" 内にある java.lang.ref.Reference の処理を行う.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // ReferenceProcessor class encapsulates the per-"collector" processing
    // of java.lang.Reference objects for GC. The interface is useful for supporting
    // a generational abstraction, in particular when there are multiple
    // generations that are being independently collected -- possibly
    // concurrently and/or incrementally.  Note, however, that the
    // ReferenceProcessor class abstracts away from a generational setting
    // by using only a heap interval (called "span" below), thus allowing
    // its use in a straightforward manner in a general, non-generational
    // setting.
    //
    // The basic idea is that each ReferenceProcessor object concerns
    // itself with ("weak") reference processing in a specific "span"
    // of the heap of interest to a specific collector. Currently,
    // the span is a convex interval of the heap, but, efficiency
    // apart, there seems to be no reason it couldn't be extended
    // (with appropriate modifications) to any "non-convex interval".
```

java.lang.ref オブジェクトの処理は, ReferenceProcessor の以下のメソッドで行われる.

* (A) ReferenceProcessor::process_discovered_references()
* (B) ReferenceProcessor::enqueue_discovered_references()

なお, この (A) や (B) の処理をマルチスレッドで行うための枠組みとして,
AbstractRefProcTaskExecutor というクラスが用意されている.
マルチスレッド化する場合は以下のようになる.

* (A) や (B) のメソッドは引数として AbstractRefProcTaskExecutor オブジェクトを受け取れるようになっており,
  呼び出された ReferenceProcessor オブジェクトの _processing_is_mt フィールドが true であれば,
  AbstractRefProcTaskExecutor::execute() による並列処理が行われる
  (逆に, 並列に行いたくないときには, この引数を NULL にして呼び出せばよい).

* 並列実行の際には, (A) や (B) の処理が
  AbstractRefProcTaskExecutor::ProcessTask クラスや
  AbstractRefProcTaskExecutor::EnqueueTask クラスのオブジェクトとして表現される.
  そして, AbstractRefProcTaskExecutor オブジェクトは「それらを受け取って並列に実行する」という役割を担う.

* なお実際には AbstractRefProcTaskExecutor クラス自体は abstract class で,
  それぞれの GC アルゴリズム毎に具体的なサブクラスが存在している.
  (AbstractRefProcTaskExecutor::ProcessTask クラスや
   AbstractRefProcTaskExecutor::EnqueueTask クラスも abstract class であり,
   GC アルゴリズム毎に具体的なサブクラスを持つ)



### クラス一覧(class list)

  * [ReferenceProcessor](#no31yNNYRw)
  * [NoRefDiscovery](#noWQWG4LcC)
  * [ReferenceProcessorSpanMutator](#noiV-RSWoF)
  * [ReferenceProcessorMTDiscoveryMutator](#noq2yIMLVN)
  * [ReferenceProcessorIsAliveMutator](#nosFTtjtWQ)
  * [ReferenceProcessorAtomicMutator](#noBWUpczY7)
  * [ReferenceProcessorMTProcMutator](#noCP38y8mR)
  * [AbstractRefProcTaskExecutor](#noUFjRpHs_)
  * [AbstractRefProcTaskExecutor::ProcessTask](#noUdQfTP1q)
  * [AbstractRefProcTaskExecutor::EnqueueTask](#noh1wmR4--)
  * [DiscoveredList](#noCQ-Qe4Pc)
  * [AlwaysAliveClosure](#no61iDfLg-)
  * [CountHandleClosure](#noGBqnAya3)
  * [RefProcEnqueueTask](#noC-wk3vbZ)
  * [DiscoveredListIterator](#noderjK_zD)
  * [RefProcPhase1Task](#noMRwHqjCv)
  * [RefProcPhase2Task](#noA4_6V0gQ)
  * [RefProcPhase3Task](#noZkQHi7_z)


---
## <a name="no31yNNYRw" id="no31yNNYRw">ReferenceProcessor</a>

### 概要(Summary)
java.lang.ref.Reference オブジェクトの処理を行うためのクラス (See: [here](no289169tf.html) for details).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    class ReferenceProcessor : public CHeapObj {
```

### 内部構造(Internal structure)
このクラスの以下のメソッドが java.lang.ref.Reference オブジェクトの処理のエントリポイントになっている.

* ReferenceProcessor::process_discovered_references()

  参照先が生きている参照オブジェクトをリストから除外する処理.

* ReferenceProcessor::enqueue_discovered_references()

  リストに残った参照オブジェクト(= 参照先が死んだので回収処理が必要な参照オブジェクト)を pending list に追加する処理.




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessor.html) for details

---
## <a name="noWQWG4LcC" id="noWQWG4LcC">NoRefDiscovery</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor を無効にするためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to disable reference discovery in
    // the scope which contains it, for given ReferenceProcessor.
    class NoRefDiscovery: StackObj {
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで disable_discovery() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      NoRefDiscovery(ReferenceProcessor* rp) : _rp(rp) {
        _was_discovering_refs = _rp->discovery_enabled();
        if (_was_discovering_refs) {
          _rp->disable_discovery();
        }
      }
    
      ~NoRefDiscovery() {
        if (_was_discovering_refs) {
          _rp->enable_discovery();
        }
      }
```




### 詳細(Details)
See: [here](../doxygen/classNoRefDiscovery.html) for details

---
## <a name="noiV-RSWoF" id="noiV-RSWoF">ReferenceProcessorSpanMutator</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor の対象範囲を変更するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to temporarily mutate the span of the
    // given ReferenceProcessor in the scope that contains it.
    class ReferenceProcessorSpanMutator: StackObj {
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで set_span() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      ReferenceProcessorSpanMutator(ReferenceProcessor* rp,
                                    MemRegion span):
        _rp(rp) {
        _saved_span = _rp->span();
        _rp->set_span(span);
      }
    
      ~ReferenceProcessorSpanMutator() {
        _rp->set_span(_saved_span);
      }
```




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessorSpanMutator.html) for details

---
## <a name="noq2yIMLVN" id="noq2yIMLVN">ReferenceProcessorMTDiscoveryMutator</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor の処理をマルチスレッドで行うかどうかの設定を変更するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to temporarily change the MT'ness of
    // reference discovery for the given ReferenceProcessor
    // in the scope that contains it.
    class ReferenceProcessorMTDiscoveryMutator: StackObj {
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで set_mt_discovery() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      ReferenceProcessorMTDiscoveryMutator(ReferenceProcessor* rp,
                                           bool mt):
        _rp(rp) {
        _saved_mt = _rp->discovery_is_mt();
        _rp->set_mt_discovery(mt);
      }
    
      ~ReferenceProcessorMTDiscoveryMutator() {
        _rp->set_mt_discovery(_saved_mt);
      }
```




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessorMTDiscoveryMutator.html) for details

---
## <a name="nosFTtjtWQ" id="nosFTtjtWQ">ReferenceProcessorIsAliveMutator</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor の _is_alive_non_header フィールドを変更するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to temporarily change the disposition
    // of the "is_alive_non_header" closure field of the
    // given ReferenceProcessor in the scope that contains it.
    class ReferenceProcessorIsAliveMutator: StackObj {
```

なお _is_alive_non_header フィールドとは, 
mark フィールドに情報を書き込まない GC アルゴリズムと併用する場合に, 
オブジェクトに GC が到達するかどうかを知るための Closure を入れておくフィールド
(現在は CMS と併用する場合にしか使われない).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      // For collectors that do not keep GC marking information
      // in the object header, this field holds a closure that
      // helps the reference processor determine the reachability
      // of an oop (the field is currently initialized to NULL for
      // all collectors but the CMS collector).
      BoolObjectClosure* _is_alive_non_header;
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで set_is_alive_non_header() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      ReferenceProcessorIsAliveMutator(ReferenceProcessor* rp,
                                       BoolObjectClosure*  cl):
        _rp(rp) {
        _saved_cl = _rp->is_alive_non_header();
        _rp->set_is_alive_non_header(cl);
      }
    
      ~ReferenceProcessorIsAliveMutator() {
        _rp->set_is_alive_non_header(_saved_cl);
      }
```




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessorIsAliveMutator.html) for details

---
## <a name="noBWUpczY7" id="noBWUpczY7">ReferenceProcessorAtomicMutator</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor の discovery_is_atomic フィールドを変更するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to temporarily change the disposition
    // of the "discovery_is_atomic" field of the
    // given ReferenceProcessor in the scope that contains it.
    class ReferenceProcessorAtomicMutator: StackObj {
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで set_atomic_discovery() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      ReferenceProcessorAtomicMutator(ReferenceProcessor* rp,
                                      bool atomic):
        _rp(rp) {
        _saved_atomic_discovery = _rp->discovery_is_atomic();
        _rp->set_atomic_discovery(atomic);
      }
    
      ~ReferenceProcessorAtomicMutator() {
        _rp->set_atomic_discovery(_saved_atomic_discovery);
      }
```




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessorAtomicMutator.html) for details

---
## <a name="noCP38y8mR" id="noCP38y8mR">ReferenceProcessorMTProcMutator</a>

### 概要(Summary)
ReferenceProcessor クラス用のユーティリティ・クラス(StackObjクラス).

ソースコード中のあるスコープの間だけ, ReferenceProcessor の _processing_is_mt フィールドを変更するためのクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // A utility class to temporarily change the MT processing
    // disposition of the given ReferenceProcessor instance
    // in the scope that contains it.
    class ReferenceProcessorMTProcMutator: StackObj {
```

### 内部構造(Internal structure)
コンストラクタ引数で指定された ReferenceProcessor オブジェクトに対し, 
コンストラクタで set_mt_processing() を呼び出し, デストラクタで元の状態に復元させている.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
      ReferenceProcessorMTProcMutator(ReferenceProcessor* rp,
                                      bool mt):
        _rp(rp) {
        _saved_mt = _rp->processing_is_mt();
        _rp->set_mt_processing(mt);
      }
    
      ~ReferenceProcessorMTProcMutator() {
        _rp->set_mt_processing(_saved_mt);
      }
```




### 詳細(Details)
See: [here](../doxygen/classReferenceProcessorMTProcMutator.html) for details

---
## <a name="noUFjRpHs_" id="noUFjRpHs_">AbstractRefProcTaskExecutor</a>

### 概要(Summary)
ReferenceProcessor の処理をマルチスレッド化するクラス (の基底クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // This class is an interface used to implement task execution for the
    // reference processing.
    class AbstractRefProcTaskExecutor {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classAbstractRefProcTaskExecutor.html) for details

---
## <a name="noUdQfTP1q" id="noUdQfTP1q">AbstractRefProcTaskExecutor::ProcessTask</a>

### 概要(Summary)
ReferenceProcessor::process_discovered_references() の処理をマルチスレッド化するクラス (の基底クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // Abstract reference processing task to execute.
    class AbstractRefProcTaskExecutor::ProcessTask {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classAbstractRefProcTaskExecutor_1_1ProcessTask.html) for details

---
## <a name="noh1wmR4--" id="noh1wmR4--">AbstractRefProcTaskExecutor::EnqueueTask</a>

### 概要(Summary)
ReferenceProcessor::enqueue_discovered_references() の処理をマルチスレッド化するクラス (の基底クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.hpp))
    // Abstract reference processing task to execute.
    class AbstractRefProcTaskExecutor::EnqueueTask {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classAbstractRefProcTaskExecutor_1_1EnqueueTask.html) for details

---
## <a name="noCQ-Qe4Pc" id="noCQ-Qe4Pc">DiscoveredList</a>

### 概要(Summary)
ReferenceProcessor クラス内で使用される補助クラス.

ReferenceProcessor::discover_reference() で見つかった結果を格納しておくためのリスト (See: [here](no289169tf.html) for details).

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    // List of discovered references.
    class DiscoveredList {
```




### 詳細(Details)
See: [here](../doxygen/classDiscoveredList.html) for details

---
## <a name="no61iDfLg-" id="no61iDfLg-">AlwaysAliveClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    #ifndef PRODUCT
```

ReferenceProcessor::count_jni_refs() 内で使用される補助クラス
(というか, ReferenceProcessor::count_jni_refs() 内で定義されているので関数外からは見えもしないが....
なお ReferenceProcessor::count_jni_refs() 自体も #ifndef PRODUCT 時にしか定義されない関数で, 
JNI Weak Global Handle の数を数えてリターンするというもの)

JNI の Weak Global Handle を辿る処理で使用される Closure.

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      class AlwaysAliveClosure: public BoolObjectClosure {
```

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/runtime/jniHandles.cpp にいたりするが
(そして処理もそっくりだが), 特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classAlwaysAliveClosure.html) for details

---
## <a name="noGBqnAya3" id="noGBqnAya3">CountHandleClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    #ifndef PRODUCT
```

ReferenceProcessor::count_jni_refs() 内で使用される補助クラス
(というか, ReferenceProcessor::count_jni_refs() 内で定義されているので関数外からは見えもしないが....
なお ReferenceProcessor::count_jni_refs() 自体も #ifndef PRODUCT 時にしか定義されない関数で, 
JNI Weak Global Handle の数を数えてリターンするというもの)

JNI の Weak Global Handle を数える処理で使用される Closure.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      class CountHandleClosure: public OopClosure {
```

### 備考(Notes)
同名のクラスが hotspot/src/share/vm/runtime/jniHandles.cpp にいたりするが
(そして処理もそっくりだが), 特に関係は無い模様.




### 詳細(Details)
See: [here](../doxygen/classCountHandleClosure.html) for details

---
## <a name="noC-wk3vbZ" id="noC-wk3vbZ">RefProcEnqueueTask</a>

### 概要(Summary)
ReferenceProcessor::enqueue_discovered_references() 内で使用される補助クラス
(より正確には, そこから呼び出される ReferenceProcessor::enqueue_discovered_reflists() 内で使用される補助クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    // Parallel enqueue task
    class RefProcEnqueueTask: public AbstractRefProcTaskExecutor::EnqueueTask {
```

### 使われ方(Usage)
AbstractRefProcTaskExecutor::EnqueueTask のサブクラス.

このオブジェクトが AbstractRefProcTaskExecutor に渡されることで
ReferenceProcessor::enqueue_discovered_references() の処理がマルチスレッド化される.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    // Enqueue references that are not made active again
    void ReferenceProcessor::enqueue_discovered_reflists(HeapWord* pending_list_addr,
      AbstractRefProcTaskExecutor* task_executor) {
      if (_processing_is_mt && task_executor != NULL) {
        // Parallel code
        RefProcEnqueueTask tsk(*this, _discoveredSoftRefs,
                               pending_list_addr, sentinel_ref(), _max_num_q);
        task_executor->execute(tsk);
      } else {
```




### 詳細(Details)
See: [here](../doxygen/classRefProcEnqueueTask.html) for details

---
## <a name="noderjK_zD" id="noderjK_zD">DiscoveredListIterator</a>

### 概要(Summary)
DiscoveredList を辿るためのイテレータクラス.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    // Iterator for the list of discovered references.
    class DiscoveredListIterator {
```




### 詳細(Details)
See: [here](../doxygen/classDiscoveredListIterator.html) for details

---
## <a name="noMRwHqjCv" id="noMRwHqjCv">RefProcPhase1Task</a>

### 概要(Summary)
ReferenceProcessor::process_discovered_references() 内で使用される補助クラス
(より正確には, そこから呼び出される ReferenceProcessor::process_discovered_reflist() 内で使用される補助クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    class RefProcPhase1Task: public AbstractRefProcTaskExecutor::ProcessTask {
```

### 使われ方(Usage)
AbstractRefProcTaskExecutor::ProcessTask のサブクラス.

このオブジェクトが AbstractRefProcTaskExecutor に渡されることで
ReferenceProcessor::process_discovered_reflist() の Phase 1 の処理がマルチスレッド化される.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      // Phase 1 (soft refs only):
      // . Traverse the list and remove any SoftReferences whose
      //   referents are not alive, but that should be kept alive for
      //   policy reasons. Keep alive the transitive closure of all
      //   such referents.
      if (policy != NULL) {
        if (mt_processing) {
          RefProcPhase1Task phase1(*this, refs_lists, policy, true /*marks_oops_alive*/);
          task_executor->execute(phase1);
        } else {
```




### 詳細(Details)
See: [here](../doxygen/classRefProcPhase1Task.html) for details

---
## <a name="noA4_6V0gQ" id="noA4_6V0gQ">RefProcPhase2Task</a>

### 概要(Summary)
ReferenceProcessor::process_discovered_references() 内で使用される補助クラス
(より正確には, そこから呼び出される ReferenceProcessor::process_discovered_reflist() 内で使用される補助クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    class RefProcPhase2Task: public AbstractRefProcTaskExecutor::ProcessTask {
```

### 使われ方(Usage)
AbstractRefProcTaskExecutor::ProcessTask のサブクラス.

このオブジェクトが AbstractRefProcTaskExecutor に渡されることで
ReferenceProcessor::process_discovered_reflist() の Phase 2 の処理がマルチスレッド化される.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      // Phase 2:
      // . Traverse the list and remove any refs whose referents are alive.
      if (mt_processing) {
        RefProcPhase2Task phase2(*this, refs_lists, !discovery_is_atomic() /*marks_oops_alive*/);
        task_executor->execute(phase2);
      } else {
```




### 詳細(Details)
See: [here](../doxygen/classRefProcPhase2Task.html) for details

---
## <a name="noZkQHi7_z" id="noZkQHi7_z">RefProcPhase3Task</a>

### 概要(Summary)
ReferenceProcessor::process_discovered_references() 内で使用される補助クラス
(より正確には, そこから呼び出される ReferenceProcessor::process_discovered_reflist() 内で使用される補助クラス).


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
    class RefProcPhase3Task: public AbstractRefProcTaskExecutor::ProcessTask {
```

### 使われ方(Usage)
AbstractRefProcTaskExecutor::ProcessTask のサブクラス.

このオブジェクトが AbstractRefProcTaskExecutor に渡されることで
ReferenceProcessor::process_discovered_reflist() の Phase 3 の処理がマルチスレッド化される.


```
    ((cite: hotspot/src/share/vm/memory/referenceProcessor.cpp))
      // Phase 3:
      // . Traverse the list and process referents as appropriate.
      if (mt_processing) {
        RefProcPhase3Task phase3(*this, refs_lists, clear_referent, true /*marks_oops_alive*/);
        task_executor->execute(phase3);
      } else {
```




### 詳細(Details)
See: [here](../doxygen/classRefProcPhase3Task.html) for details

---
