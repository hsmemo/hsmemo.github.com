---
layout: default
title: G1RemSet クラス関連のクラス (G1RemSet, CountNonCleanMemRegionClosure, UpdateRSOopClosure, UpdateRSetImmediate, UpdateRSOrPushRefOopClosure, 及びそれらの補助クラス(IntoCSOopClosure, VerifyRSCleanCardOopClosure, ScanRSClosure, RefineRecordRefsIntoCSCardTableEntryClosure, PrintRSClosure, CountRSSizeClosure, cleanUpIteratorsClosure, UpdateRSetCardTableEntryIntoCSetClosure, ScrubRSClosure, TriggerClosure, InvokeIfNotTriggeredClosure, Mux2Closure, HRRSStatsIter, PrintRSThreadVTimeClosure))
---
[Top](../index.html)

#### G1RemSet クラス関連のクラス (G1RemSet, CountNonCleanMemRegionClosure, UpdateRSOopClosure, UpdateRSetImmediate, UpdateRSOrPushRefOopClosure, 及びそれらの補助クラス(IntoCSOopClosure, VerifyRSCleanCardOopClosure, ScanRSClosure, RefineRecordRefsIntoCSCardTableEntryClosure, PrintRSClosure, CountRSSizeClosure, cleanUpIteratorsClosure, UpdateRSetCardTableEntryIntoCSetClosure, ScrubRSClosure, TriggerClosure, InvokeIfNotTriggeredClosure, Mux2Closure, HRRSStatsIter, PrintRSThreadVTimeClosure))

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Remembered Set の処理を補佐するためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    // A G1RemSet provides ways of iterating over pointers into a selected
    // collection set.
```


### クラス一覧(class list)

  * [G1RemSet](#noXHkr9Oib)
  * [CountNonCleanMemRegionClosure](#nowr7334Xs)
  * [UpdateRSOopClosure](#noFvrIWkMR)
  * [UpdateRSetImmediate](#no_thrpgwH)
  * [UpdateRSOrPushRefOopClosure](#noDBBd4JAw)
  * [IntoCSOopClosure](#noUbEBjNjJ)
  * [VerifyRSCleanCardOopClosure](#nosOrUCZo9)
  * [ScanRSClosure](#noDzQiPULO)
  * [RefineRecordRefsIntoCSCardTableEntryClosure](#notsDPHFi-)
  * [PrintRSClosure](#nojCGdcAN2)
  * [CountRSSizeClosure](#no7rwlRuGN)
  * [cleanUpIteratorsClosure](#nojdKiN78h)
  * [UpdateRSetCardTableEntryIntoCSetClosure](#noV13HPxkU)
  * [ScrubRSClosure](#nodqg5dAjV)
  * [TriggerClosure](#noDoAoNRVB)
  * [InvokeIfNotTriggeredClosure](#noO6ft8kbX)
  * [Mux2Closure](#noNMsfvbwL)
  * [HRRSStatsIter](#noKGKw6The)
  * [PrintRSThreadVTimeClosure](#noz5ke7iab)


---
## <a name="noXHkr9Oib" id="noXHkr9Oib">G1RemSet</a>

### 概要(Summary)
G1CollectedHeap の Remembered Set 情報 (HeapRegionRemSet オブジェクト) を処理するためのクラス
(See: HeapRegionRemSet).

Remembered Set 処理用のメソッドを提供している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    // A G1RemSet in which each heap region has a rem set that records the
    // external heap references into it.  Uses a mod ref bs to track updates,
    // so that they can be used to update the individual region remsets.
    
    class G1RemSet: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _g1_rem_set フィールドに(のみ)格納されている

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* G1CollectedHeap * 	_g1
* unsigned 	_conc_refine_cards

* CardTableModRefBS * 	_ct_bs
  
  SharedHeap::_rem_set に入っている CardTableRS の 
  _ct_bs フィールドの値(G1SATBCardTableLoggingModRefBS オブジェクト)と同じオブジェクト

  (CardTableRS::CardTableRS() で作成した G1SATBCardTableLoggingModRefBS オブジェクトが CardTableRS::_bs にセットされ, 
   G1CollectedHeap::initialize() で生成された CardTableRS が SharedHeap::_rem_set にセットされる.
   その後, _barrier_set に, CardTableRS 中の _bs の値 (= 作成した G1SATBCardTableLoggingModRefBS) がセットされる.
   これが _mr_bs にもコピーされ, 最終的に G1RemSet のコンストラクタに渡される)

* SubTasksDone * 	_seq_task
* G1CollectorPolicy * 	_g1p
* ConcurrentG1Refine * 	_cg1r
* size_t * 	_cards_scanned
* size_t 	_total_cards_scanned
* OopsInHeapRegionClosure ** 	_cset_rs_update_cl


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
      G1CollectedHeap* _g1;
      unsigned _conc_refine_cards;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
      CardTableModRefBS*             _ct_bs;
      SubTasksDone*                  _seq_task;
      G1CollectorPolicy* _g1p;
    
      ConcurrentG1Refine* _cg1r;
    
      size_t*             _cards_scanned;
      size_t              _total_cards_scanned;
    
      // Used for caching the closure that is responsible for scanning
      // references into the collection set.
      OopsInHeapRegionClosure** _cset_rs_update_cl;
```




### 詳細(Details)
See: [here](../doxygen/classG1RemSet.html) for details

---
## <a name="nowr7334Xs" id="nowr7334Xs">CountNonCleanMemRegionClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス(?? #TODO) (関連する develop オプションが指定されている場合にのみ使用される) 
(See: G1RSLogCheckCardTable).

Barrier Set (CardTableModRefBS) 中にある dirty なカードの数を数えるための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    class CountNonCleanMemRegionClosure: public MemRegionClosure {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 調べたい CardTableModRefBS に対し, このクロージャを引数として CardTableModRefBS::mod_card_iterate() を呼び出す.
2. 数えた結果を CountNonCleanMemRegionClosure::n() で取得する.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* G1CollectedHeap::check_ct_logs_at_safepoint() 内
  
  (が, この関数自体が使われていないような...?? #TODO)

* G1RemSet::updateRS() 

  ただし, develop オプションである G1RSLogCheckCardTable が変更されている場合(= false の場合)にのみ使用される.
  また, 使用用途は guarantee() でのチェックに用いるだけ (verify 用途).

### 内部構造(Internal structure)
CountNonCleanMemRegionClosure::do_MemRegion() が呼ばれる度に 
CountNonCleanMemRegionClosure::_n フィールドが対応する card 個数分だけ増加する
(その MemRegion が g1 のヒープ外なら増加しないが...).

#### 参考(for your information): CountNonCleanMemRegionClosure::do_MemRegion()
See: [here](no3420Tdu.html) for details



### 詳細(Details)
See: [here](../doxygen/classCountNonCleanMemRegionClosure.html) for details

---
## <a name="noFvrIWkMR" id="noFvrIWkMR">UpdateRSOopClosure</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される補助クラス (See: [here](no2935ATn.html) for details).

RebuildRSOutOfRegionClosure クラス内で使用される補助クラス.
(Major GC の処理後に) 全ての HeapRegion の Remembered Set 情報を作成し直すための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    class UpdateRSOopClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 RebuildRSOutOfRegionClosure オブジェクトの _cl フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(RebuildRSOutOfRegionClosure クラスの _cl フィールドは, ポインタ型ではなく実体なので,
 RebuildRSOutOfRegionClosure オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
RebuildRSOutOfRegionClosure::doHeapRegion() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classUpdateRSOopClosure.html) for details

---
## <a name="no_thrpgwH" id="no_thrpgwH">UpdateRSetImmediate</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC がメモリ不足で失敗した場合に, 
コピーできなかったオブジェクトについて, Remembered Set 情報を作成し直す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    class UpdateRSetImmediate: public OopsInHeapRegionClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている
(ただし G1CollectedHeap::remove_self_forwarding_pointers() の方については, 
 develop オプションである G1DeferredRSUpdate が変更されている場合(= false の場合)にのみ使用される.
 G1DeferredRSUpdate オプションが true の場合には, 代わりに UpdateRSetDeferred クラスが使用される).

* UpdateRSetCardTableEntryIntoCSetClosure::do_card_ptr() 内
* G1CollectedHeap::remove_self_forwarding_pointers() 内
  



### 詳細(Details)
See: [here](../doxygen/classUpdateRSetImmediate.html) for details

---
## <a name="noDBBd4JAw" id="noDBBd4JAw">UpdateRSOrPushRefOopClosure</a>

### 概要(Summary)
G1RemSet の処理で使用される補助クラス. 

Remembered Set の修正処理を行うための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
    class UpdateRSOrPushRefOopClosure: public OopClosure {
```

### 使われ方(Usage)
G1RemSet::concurrentRefineOneCard_impl() 内で(のみ)使用されている (See: [here](no3420WIS.html) for details).




### 詳細(Details)
See: [here](../doxygen/classUpdateRSOrPushRefOopClosure.html) for details

---
## <a name="noUbEBjNjJ" id="noUbEBjNjJ">IntoCSOopClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class IntoCSOopClosure: public OopsInHeapRegionClosure {
```



### 詳細(Details)
See: [here](../doxygen/classIntoCSOopClosure.html) for details

---
## <a name="nosOrUCZo9" id="nosOrUCZo9">VerifyRSCleanCardOopClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class VerifyRSCleanCardOopClosure: public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classVerifyRSCleanCardOopClosure.html) for details

---
## <a name="noDzQiPULO" id="noDzQiPULO">ScanRSClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Collection Set 外から Collection Set 内を指しているポインタを処理する Closure.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class ScanRSClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
G1RemSet::scanRS() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classScanRSClosure.html) for details

---
## <a name="notsDPHFi-" id="notsDPHFi-">RefineRecordRefsIntoCSCardTableEntryClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Collection Set 外から Collection Set 内を指しているポインタを発見し, Remembered Set を更新するための Closure.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    // Closure used for updating RSets and recording references that
    // point into the collection set. Only called during an
    // evacuation pause.
    
    class RefineRecordRefsIntoCSCardTableEntryClosure: public CardTableEntryClosure {
```

### 使われ方(Usage)
G1RemSet::updateRS() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classRefineRecordRefsIntoCSCardTableEntryClosure.html) for details

---
## <a name="nojCGdcAN2" id="nojCGdcAN2">PrintRSClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

処理対象の HeapRegion の Remembered Set に関する情報を出力する Closure クラス


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    #ifndef PRODUCT
    class PrintRSClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている
(ただし, #if G1_REM_SET_LOGGING 時にしか使用されない).

* G1RemSet::prepare_for_oops_into_collection_set_do()
* G1RemSet::cleanup_after_oops_into_collection_set_do()




### 詳細(Details)
See: [here](../doxygen/classPrintRSClosure.html) for details

---
## <a name="no7rwlRuGN" id="no7rwlRuGN">CountRSSizeClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: G1RSCountHisto).

各 HeapRegion の Remembered Set に関する情報を収集する Closure クラス
(Remembered Set として使用しているメモリ量(合計値,最大値,ヒストグラム情報), 
 最もメモリを使用している Remembered Set に対応する HeapRegion, 等).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class CountRSSizeClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1RemSet::oops_into_collection_set_do() 内で(のみ)使用されている
(ただし, G1RSCountHisto オプションが指定されている時にしか使用されない).




### 詳細(Details)
See: [here](../doxygen/classCountRSSizeClosure.html) for details

---
## <a name="nojdKiN78h" id="nojdKiN78h">cleanUpIteratorsClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC がメモリ不足で失敗した場合に, 
Collection Set に入っている HeapRegion の 
Remembered Set の状態 (HeapRegionRemSet::_iter_state) をリセットするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class cleanUpIteratorsClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
G1RemSet::cleanup_after_oops_into_collection_set_do() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::evacuate_collection_set()
-&gt; G1RemSet::cleanup_after_oops_into_collection_set_do()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classcleanUpIteratorsClosure.html) for details

---
## <a name="noV13HPxkU" id="noV13HPxkU">UpdateRSetCardTableEntryIntoCSetClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC がメモリ不足で失敗した場合に, 
Collection Set 内を指している HeapRegion の Remembered Set を修正するための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    // This closure, applied to a DirtyCardQueueSet, is used to immediately
    // update the RSets for the regions in the CSet. For each card it iterates
    // through the oops which coincide with that card. It scans the reference
    // fields in each oop; when it finds an oop that points into the collection
    // set, the RSet for the region containing the referenced object is updated.
    class UpdateRSetCardTableEntryIntoCSetClosure: public CardTableEntryClosure {
```

### 使われ方(Usage)
G1RemSet::cleanup_after_oops_into_collection_set_do() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::evacuate_collection_set()
-&gt; G1RemSet::cleanup_after_oops_into_collection_set_do()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classUpdateRSetCardTableEntryIntoCSetClosure.html) for details

---
## <a name="nodqg5dAjV" id="nodqg5dAjV">ScrubRSClosure</a>

### 概要(Summary)
G1ParScrubRemSetTask クラス内で使用される補助クラス.

生きているオブジェクトがいない HeapRegion について, 
そこに付いている Remembered Set 用のデータ構造を解放する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class ScrubRSClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* G1RemSet::scrub()
* G1RemSet::scrub_par()

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* G1ParScrubRemSetTask::work()
  -&gt; G1RemSet::scrub()

* G1ParScrubRemSetTask::work()
  -&gt; G1RemSet::scrub_par()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classScrubRSClosure.html) for details

---
## <a name="noDoAoNRVB" id="noDoAoNRVB">TriggerClosure</a>

### 概要(Summary)
G1RemSet の処理で使用される補助クラス. 

InvokeIfNotTriggeredClosure と組み合わせて使用される Closure クラス
(See: InvokeIfNotTriggeredClosure).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class TriggerClosure : public OopClosure {
```

### 使われ方(Usage)
G1RemSet::concurrentRefineOneCard_impl() 内で(のみ)使用されている (See: [here](no3420WIS.html) for details).

### 内部構造(Internal structure)
TriggerClosure::value() というメソッドを備えている.
このメソッドは, 一度でも do_oop() が呼び出されたことがあれば true, そうでなければ false を返す.

#### 参考(for your information): TriggerClosure::TriggerClosure()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
      TriggerClosure() : _trigger(false) { }
```

#### 参考(for your information): TriggerClosure::do_oop()
See: [here](no3420FgJ.html) for details
#### 参考(for your information): TriggerClosure::do_oop_nv()
See: [here](no3420SqP.html) for details
#### 参考(for your information): TriggerClosure::value()
See: [here](no3420f0V.html) for details



### 詳細(Details)
See: [here](../doxygen/classTriggerClosure.html) for details

---
## <a name="noO6ft8kbX" id="noO6ft8kbX">InvokeIfNotTriggeredClosure</a>

### 概要(Summary)
G1RemSet の処理で使用される補助クラス. 

他の OopClosure と組み合わせて使用される Closure クラス.
「指定した TriggerClosure が false を返す場合にのみ OopClosure を適用したい」という場合に使用される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class InvokeIfNotTriggeredClosure: public OopClosure {
```

### 使われ方(Usage)
G1RemSet::concurrentRefineOneCard_impl() 内で(のみ)使用されている (See: [here](no3420WIS.html) for details).

### 内部構造(Internal structure)
コンストラクタで TriggerClosure と OopClosure を受け取る.
内部の処理では, TriggerClosure::value() が false を返した場合にのみ,
指定された OopClosure の適用を行う.

#### 参考(for your information): InvokeIfNotTriggeredClosure::InvokeIfNotTriggeredClosure()
See: [here](no34205Bu.html) for details
#### 参考(for your information): InvokeIfNotTriggeredClosure::do_oop()
See: [here](no3420GM0.html) for details
#### 参考(for your information): InvokeIfNotTriggeredClosure::do_oop_nv()
See: [here](no34204VD.html) for details



### 詳細(Details)
See: [here](../doxygen/classInvokeIfNotTriggeredClosure.html) for details

---
## <a name="noNMsfvbwL" id="noNMsfvbwL">Mux2Closure</a>

### 概要(Summary)
G1RemSet の処理で使用される補助クラス. 

2つの OopClosure を合成した Closure を作成する
(といっても関数合成ではなく同じ引数に対して2つを逐次的に適用するだけだけど).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class Mux2Closure : public OopClosure {
```

### 使われ方(Usage)
G1RemSet::concurrentRefineOneCard_impl() 内で(のみ)使用されている (See: [here](no3420WIS.html) for details).

### 内部構造(Internal structure)
コンストラクタで 2つの OopClosure を受け取る.
内部の処理では, 単に 2つの OopClosure を順に適用するだけ.

#### 参考(for your information): Mux2Closure::Mux2Closure()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
      Mux2Closure(OopClosure *c1, OopClosure *c2) : _c1(c1), _c2(c2) { }
```

#### 参考(for your information): Mux2Closure::do_oop()
See: [here](no3420fth.html) for details
#### 参考(for your information): Mux2Closure::do_oop_nv()
See: [here](no3420s3n.html) for details



### 詳細(Details)
See: [here](../doxygen/classMux2Closure.html) for details

---
## <a name="noKGKw6The" id="noKGKw6The">HRRSStatsIter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: ExitAfterGCNum).

各 HeapRegion の Remembered Set に関する情報を収集する Closure クラス
(Remembered Set として使用しているメモリ量(合計値,最大値), 最もメモリを使用している Remembered Set に対応する HeapRegion, 等).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class HRRSStatsIter: public HeapRegionClosure {
```

### 使われ方(Usage)
G1RemSet::print_summary_info() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

(なお, このクラスは (ExitAfterGCNum オプションに加えて) 
 diagnostic オプションである G1SummarizeRSetStats も設定されている場合にしか使用されない)

<div class="flow-abst"><pre>
G1CollectedHeap::do_collection_pause_at_safepoint()
-&gt; G1CollectedHeap::print_tracing_info() (&lt;= ExitAfterGCNum オプションが指定されている場合にのみ呼び出す)
   -&gt; G1RemSet::print_summary_info()     (&lt;= G1SummarizeRSetStats オプションが指定されている場合にのみ呼び出す)
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classHRRSStatsIter.html) for details

---
## <a name="noz5ke7iab" id="noz5ke7iab">PrintRSThreadVTimeClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: ExitAfterGCNum).

処理対象の ConcurrentG1RefineThread の稼働時間(生成されてからの時間)を出力する
(See: ConcurrentG1RefineThread::vtime_accum(), ConcurrentG1RefineThread::run()).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp))
    class PrintRSThreadVTimeClosure : public ThreadClosure {
```

### 使われ方(Usage)
G1RemSet::print_summary_info() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

(なお, このクラスは (ExitAfterGCNum オプションに加えて) 
 diagnostic オプションである G1SummarizeRSetStats も設定されている場合にしか使用されない)

<div class="flow-abst"><pre>
G1CollectedHeap::do_collection_pause_at_safepoint()
-&gt; G1CollectedHeap::print_tracing_info() (&lt;= ExitAfterGCNum オプションが指定されている場合にのみ呼び出す)
   -&gt; G1RemSet::print_summary_info()     (&lt;= G1SummarizeRSetStats オプションが指定されている場合にのみ呼び出す)
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classPrintRSThreadVTimeClosure.html) for details

---
