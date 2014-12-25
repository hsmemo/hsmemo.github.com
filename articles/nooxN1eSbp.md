---
layout: default
title: G1GC の処理で使用される Closure クラス (OopsInHeapRegionClosure, G1ParClosureSuper, G1ParPushHeapRSClosure, G1ParScanClosure, G1ParScanPartialArrayClosure, G1ParCopyHelper, G1ParCopyClosure, FilterIntoCSClosure, FilterInHeapRegionAndIntoCSClosure, FilterAndMarkInHeapRegionAndIntoCSClosure, FilterOutOfRegionClosure)
---
[Top](../index.html)

#### G1GC の処理で使用される Closure クラス (OopsInHeapRegionClosure, G1ParClosureSuper, G1ParPushHeapRSClosure, G1ParScanClosure, G1ParScanPartialArrayClosure, G1ParCopyHelper, G1ParCopyClosure, FilterIntoCSClosure, FilterInHeapRegionAndIntoCSClosure, FilterAndMarkInHeapRegionAndIntoCSClosure, FilterOutOfRegionClosure)

これらは, G1CollectedHeap の Garbage Collection 処理で使用される補助クラス(Closure クラス).

なお, これらのクラスは以下のような継承関係を持つ.

  * OopsInGenClosure  (<= これは別ファイルで定義されているクラス)
      * OopsInHeapRegionClosure
          * G1ParClosureSuper
              * G1ParPushHeapRSClosure
              * G1ParScanClosure
              * G1ParScanPartialArrayClosure
              * G1ParCopyHelper
                  * G1ParCopyClosure
          * FilterInHeapRegionAndIntoCSClosure
          * FilterAndMarkInHeapRegionAndIntoCSClosure

  * OopClosure       (<= これは別ファイルで定義されているクラス)
      * FilterIntoCSClosure
      * FilterOutOfRegionClosure



### クラス一覧(class list)

  * [OopsInHeapRegionClosure](#noA2tsYz7c)
  * [G1ParClosureSuper](#nolpIC29g4)
  * [G1ParPushHeapRSClosure](#noQGChyNwv)
  * [G1ParScanClosure](#nodnY4srO9)
  * [G1ParScanPartialArrayClosure](#noBQ6QSfgm)
  * [G1ParCopyHelper](#novhYnR7ll)
  * [G1ParCopyClosure](#nodmIcMERt)
  * [FilterIntoCSClosure](#nof1M-_xdG)
  * [FilterInHeapRegionAndIntoCSClosure](#noOdUT5elX)
  * [FilterAndMarkInHeapRegionAndIntoCSClosure](#noCvMJQ7Is)
  * [FilterOutOfRegionClosure](#novOHpEmhk)


---
## <a name="noA2tsYz7c" id="noA2tsYz7c">OopsInHeapRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

指定された HeapRegion 内の oop をスキャンする Closure クラスの基底クラス
(OopsInGenClosure の HeapRegion 版といった感じ).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    // A class that scans oops in a given heap region (much as OopsInGenClosure
    // scans oops in a generation.)
    class OopsInHeapRegionClosure: public OopsInGenClosure {
```




### 詳細(Details)
See: [here](../doxygen/classOopsInHeapRegionClosure.html) for details

---
## <a name="nolpIC29g4" id="nolpIC29g4">G1ParClosureSuper</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラスの基底クラス
(より正確には G1ParTask::work() 内で使用される Closure クラスの基底クラス)
(See: [here](no2935YzN.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class G1ParClosureSuper : public OopsInHeapRegionClosure {
```

### 内部構造(Internal structure)
スーパークラスである OopsInHeapRegionClosure クラスのフィールドに加えて, 以下のようなフィールドを持つ.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      G1CollectedHeap* _g1;
      G1RemSet* _g1_rem;
      ConcurrentMark* _cm;
      G1ParScanThreadState* _par_scan_state;
```




### 詳細(Details)
See: [here](../doxygen/classG1ParClosureSuper.html) for details

---
## <a name="noQGChyNwv" id="noQGChyNwv">G1ParPushHeapRSClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

処理対象のポインタが collection set 内を差していれば,
コンストラクタに渡された G1ParScanThreadState オブジェクト内のキューに登録する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class G1ParPushHeapRSClosure : public G1ParClosureSuper {
```

### 使われ方(Usage)
G1ParTask::work() 内で(のみ)使用されている
(実際に使用されるのは, そこから呼び出される G1RemSet::oops_into_collection_set_do() の中)
(See: [here](no2935YzN.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1ParPushHeapRSClosure.html) for details

---
## <a name="nodnY4srO9" id="nodnY4srO9">G1ParScanClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

オブジェクトのコピー後に, そのオブジェクト内のフィールドに対して処理を行う Closure.

* 処理対象のポインタが collection set 内を差していれば,
  コンストラクタに渡された G1ParScanThreadState オブジェクト内のキューに登録する.
* ポインタが collection set 内を差していなければ, 
  Remembered Set(G1RemSet オブジェクト) を更新する.

(G1ParPushHeapRSClosure に似ているが,
 collection set 内を差していない場合の処理もあるという点が違う.)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp))
    // This closure is applied to the fields of the objects that have just been copied.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class G1ParScanClosure : public G1ParClosureSuper {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている
(現在は「他の Closure クラスから間接的に呼び出される」という形でのみ使用されている)
(See: [here](no2935YzN.html) for details).

* G1ParCopyHelper::copy_to_survivor_space()
* G1ParScanPartialArrayClosure::do_oop_nv()

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* G1ParScanPartialArrayClosure の _scanner フィールド
* G1ParCopyClosure の _scanner フィールド (これがスーパークラスである G1ParCopyHelper のメソッドから使用される)
* G1ParCopyHelper の _scanner フィールド (ただし, このフィールドはポインタフィールド. 実体は, そのオブジェクトの G1ParCopyClosure::_scanner フィールドのものと同じ (See: G1ParCopyClosure::G1ParCopyClosure()))




### 詳細(Details)
See: [here](../doxygen/classG1ParScanClosure.html) for details

---
## <a name="noBQ6QSfgm" id="noBQ6QSfgm">G1ParScanPartialArrayClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

ポインタの配列を処理する際に, 少しずつ処理を行う(= 配列中の一部だけを taskqueue に入れ, 残りを遅延評価する)場合の処理を実装した Closure
(より具体的に言うと, 処理途中のポインタ配列を受け取り, 
 その中からある程度のポインタを取り出して taskqueue に入れ, 
 残りを pending 状態のポインタ配列として残す Closure).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class G1ParScanPartialArrayClosure : public G1ParClosureSuper {
```

### 使われ方(Usage)
G1ParTask::work() 内で(のみ)使用されている 
(実際に使用されるのは, そこから呼び出される G1ParScanThreadState::deal_with_reference() の中)
(See: [here](no2935YzN.html) for details).




### 詳細(Details)
See: [here](../doxygen/classG1ParScanPartialArrayClosure.html) for details

---
## <a name="novhYnR7ll" id="novhYnR7ll">G1ParCopyHelper</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス(の基底クラス) (See: [here](no2935YzN.html) for details).

オブジェクトのコピー処理を行う.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class G1ParCopyHelper : public G1ParClosureSuper {
```

### 内部構造(Internal structure)
内部には, Copy 処理のための以下のメソッドが定義されている (See: [here](no2935YzN.html) for details).

* G1ParCopyHelper::mark_forwardee()
  
  Initial Marking Pause の処理で使用されるメソッド.
  (Concurrent Marking 処理のために) マークビットに live だと印を付ける
  (詳細は ConcurrentMark::grayRoot() の宣言部のコメント参照).

* G1ParCopyHelper::copy_to_survivor_space()
  
  処理対象のオブジェクトをコピーするメソッド.
  

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      template <class T> void mark_forwardee(T* p);
      oop copy_to_survivor_space(oop obj);
```




### 詳細(Details)
See: [here](../doxygen/classG1ParCopyHelper.html) for details

---
## <a name="nodmIcMERt" id="nodmIcMERt">G1ParCopyClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

G1ParCopyHelper クラスの具象サブクラス.
G1CollectedHeap の Minor GC における実際のコピー処理を行う
(といっても, 実際のコピー処理はスーパークラスの G1ParCopyHelper に丸投げだったりするが...).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    template<bool do_gen_barrier, G1Barrier barrier,
             bool do_mark_forwardee>
    class G1ParCopyClosure : public G1ParCopyHelper {
```

なお以下のような型も使われるが, これは G1ParCopyClosure の別名
(G1ParCopyClosure はテンプレート引数によって 2*3*2 = 12 通りのクラスが作れる. それらに対して便利な別名が定義されている).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1_specialized_oop_closures.hpp))
    typedef G1ParCopyClosure<false, G1BarrierEvac, false> G1ParScanHeapEvacClosure;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    typedef G1ParCopyClosure<false, G1BarrierNone, false> G1ParScanExtRootClosure;
    typedef G1ParCopyClosure<true,  G1BarrierNone, false> G1ParScanPermClosure;
    typedef G1ParCopyClosure<false, G1BarrierRS,   false> G1ParScanHeapRSClosure;
    typedef G1ParCopyClosure<false, G1BarrierNone, true> G1ParScanAndMarkExtRootClosure;
    typedef G1ParCopyClosure<true,  G1BarrierNone, true> G1ParScanAndMarkPermClosure;
    typedef G1ParCopyClosure<false, G1BarrierRS,   true> G1ParScanAndMarkHeapRSClosure;
    
    // This is the only case when we set skip_cset_test. Basically, this
    // closure is (should?) only be called directly while we're draining
    // the overflow and task queues. In that case we know that the
    // reference in question points into the collection set, otherwise we
    // would not have pushed it on the queue. The following is defined in
    // g1_specialized_oop_closures.hpp.
    // typedef G1ParCopyClosure<false, G1BarrierEvac, false, true> G1ParScanHeapEvacClosure;
    // We need a separate closure to handle references during evacuation
    // failure processing, as we cannot asume that the reference already
    // points into the collection set (like G1ParScanHeapEvacClosure does).
    typedef G1ParCopyClosure<false, G1BarrierEvac, false> G1ParScanHeapEvacFailureClosure;
```


### 使われ方(Usage)
G1ParTask::work() 内で(のみ)使用されている (See: [here](no2935YzN.html) for details).

### 内部構造(Internal structure)
template 引数の意味は以下の通り. なお, barrier フィールドの型である G1Barrier は以下のような enum.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1_specialized_oop_closures.hpp))
    enum G1Barrier {
      G1BarrierNone, G1BarrierRS, G1BarrierEvac
    };
```


  * bool do_gen_barrier

    do_oop() 時に OopsInGenClosure::par_do_barrier() を呼び出すかどうか. true なら呼び出す.

  * G1Barrier barrier

    do_oop() 時に Remembered Set を更新するかどうか.

    * G1BarrierNone なら, 
      更新処理をしない.
    * G1BarrierRS なら, 
      collection set 内を指しているポインタに関してだけ Remembered Set を更新する. (備考も参照)
    * G1BarrierEvac なら, 
      ポインタが NULL でなければ (collection set 内を指しているかいないかに関わらず) Remembered Set を更新する.
      (G1ParScanHeapEvacFailureClosure の typedef 箇所にあるコメントによると,
       task queue の中のポインタを処理する場合には
       (task queue の中に入っているということから) collection set を指しているのは自明だから, とのこと) (備考も参照)

  * bool do_mark_forwardee

    do_oop() 時に Initial Marking Pause 処理を行うかどうか (= G1ParCopyHelper::mark_forwardee() を呼び出すかどうか). 
    true なら呼び出す.

### 備考(Notes)
なお, barrier フィールドの値として G1BarrierRS を使用するのは以下の別名時のみ
(しかし, これらは使われている形跡が無いんだが... 一応 G1ParTask::work() 内で生成はされてるが...).

* G1ParScanHeapRSClosure
* G1ParScanAndMarkHeapRSClosure

また, G1BarrierEvac を使用するのは以下の別名時のみ

* G1ParScanHeapEvacClosure
  
  G1ParTask::work() 内で生成され, G1ParScanThreadState::deal_with_reference() 内で使用されている.

* G1ParScanHeapEvacFailureClosure

  G1ParTask::work() 内で生成され, G1ParCopyHelper::copy_to_survivor_space() 内で使用されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      template <class T> void deal_with_reference(T* ref_to_scan) {
    ...
          _evac_cl->set_region(r);
          _evac_cl->do_oop_nv(ref_to_scan);
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    oop G1ParCopyHelper::copy_to_survivor_space(oop old) {
    ...
      if (obj_ptr == NULL) {
        // This will either forward-to-self, or detect that someone else has
        // installed a forwarding pointer.
        OopsInHeapRegionClosure* cl = _par_scan_state->evac_failure_closure();
        return _g1->handle_evacuation_failure_par(cl, old);
      }
```




### 詳細(Details)
See: [here](../doxygen/classG1ParCopyClosure.html) for details

---
## <a name="nof1M-_xdG" id="nof1M-_xdG">FilterIntoCSClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

他の OopClosure と組み合わせて使用される Closure クラス.
ある OopClosure オブジェクトを基にして
「処理対象のポインタが collection set 内を差している場合にのみ, 
その OopClosure の処理を実行する OopClosure オブジェクト」を生成する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class FilterIntoCSClosure: public OopClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* UpdateRSetCardTableEntryIntoCSetClosure::do_card_ptr() 内
* G1RemSet::concurrentRefineOneCard_impl() 内
* HeapRegionDCTOC::walk_mem_region_with_cl() 内

### 内部構造(Internal structure)
コンストラクタ引数で OopClosure が指定され, 
それが FilterIntoCSClosure::do_oop_nv() 内で使用される.

#### 参考(for your information): FilterIntoCSClosure::FilterIntoCSClosure()
See: [here](no34207Wu.html) for details
#### 参考(for your information): FilterIntoCSClosure::do_oop()
See: [here](no3420U4b.html) for details
#### 参考(for your information): FilterIntoCSClosure::do_oop_nv()
See: [here](no3420hCi.html) for details
### 備考(Notes)
コメントによると, 
FILTERINTOCSCLOSURE_DOHISTOGRAMCOUNT の処理はパフォーマンス上クリティカルなので, 
コンパイル時に on/off するように, とのこと (そして現状ではデフォルトは off).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp))
    // This must a ifdef'ed because the counting it controls is in a
    // perf-critical inner loop.
    #define FILTERINTOCSCLOSURE_DOHISTOGRAMCOUNT 0
```




### 詳細(Details)
See: [here](../doxygen/classFilterIntoCSClosure.html) for details

---
## <a name="noOdUT5elX" id="noOdUT5elX">FilterInHeapRegionAndIntoCSClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

FilterIntoCSClosure クラスに類似したクラス.
違いは, OopClosure ではなく OopsInHeapRegionClosure をラップする点.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class FilterInHeapRegionAndIntoCSClosure : public OopsInHeapRegionClosure {
```

### 内部構造(Internal structure)
コンストラクタ引数で OopsInHeapRegionClosure が指定され, 
それが FilterInHeapRegionAndIntoCSClosure::do_oop_nv() 内で使用される.

#### 参考(for your information): FilterInHeapRegionAndIntoCSClosure::FilterInHeapRegionAndIntoCSClosure()
See: [here](no34206qD.html) for details
#### 参考(for your information): FilterInHeapRegionAndIntoCSClosure::do_oop()
FilterInHeapRegionAndIntoCSClosure::do_oop_nv() を呼び出すだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      virtual void do_oop(oop* p) { do_oop_nv(p); }
      virtual void do_oop(narrowOop* p) { do_oop_nv(p); }
```

#### 参考(for your information): FilterInHeapRegionAndIntoCSClosure::do_oop_nv()
See: [here](no3420U_P.html) for details



### 詳細(Details)
See: [here](../doxygen/classFilterInHeapRegionAndIntoCSClosure.html) for details

---
## <a name="noCvMJQ7Is" id="noCvMJQ7Is">FilterAndMarkInHeapRegionAndIntoCSClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

FilterInHeapRegionAndIntoCSClosure クラスに類似したクラス.
違いは, ポインタの差し先が collection set 内ではない場合に,
もし差し先が young region でもなければ ConcurrentMark::grayRoot() を呼び出す点.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class FilterAndMarkInHeapRegionAndIntoCSClosure : public OopsInHeapRegionClosure {
```

### 内部構造(Internal structure)
コンストラクタ引数で OopsInHeapRegionClosure が指定され, 
それが FilterAndMarkInHeapRegionAndIntoCSClosure::do_oop_nv() 内で使用される.

#### 参考(for your information): FilterAndMarkInHeapRegionAndIntoCSClosure::FilterAndMarkInHeapRegionAndIntoCSClosure()
See: [here](no3420uTc.html) for details
#### 参考(for your information): FilterAndMarkInHeapRegionAndIntoCSClosure::do_oop()
FilterAndMarkInHeapRegionAndIntoCSClosure::do_oop_nv() を呼び出すだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      virtual void do_oop(oop* p) { do_oop_nv(p); }
      virtual void do_oop(narrowOop* p) { do_oop_nv(p); }
```

#### 参考(for your information): FilterAndMarkInHeapRegionAndIntoCSClosure::do_oop_nv()
See: [here](no3420Ioo.html) for details



### 詳細(Details)
See: [here](../doxygen/classFilterAndMarkInHeapRegionAndIntoCSClosure.html) for details

---
## <a name="novOHpEmhk" id="novOHpEmhk">FilterOutOfRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

他の OopClosure と組み合わせて使用される Closure クラス.
ある OopClosure オブジェクトを基にして
「処理対象のポインタが指定されたアドレス範囲 [bottom, end) の外を指している場合にのみ, 
その OopClosure の処理を行う OopClosure オブジェクト」を生成する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
    class FilterOutOfRegionClosure: public OopClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* UpdateRSetCardTableEntryIntoCSetClosure::do_card_ptr() 内
* G1RemSet::concurrentRefineOneCard_impl() 内
* HeapRegionDCTOC::walk_mem_region_with_cl() 内

### 内部構造(Internal structure)
コンストラクタ引数で OopClosure が指定され, 
それが FilterOutOfRegionClosure::do_oop_nv() 内で使用される.

#### 参考(for your information): FilterOutOfRegionClosure::FilterOutOfRegionClosure()
See: [here](no3420UGE.html) for details
#### 参考(for your information): FilterOutOfRegionClosure::do_oop()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      virtual void do_oop(oop* p) { do_oop_nv(p); }
      virtual void do_oop(narrowOop* p) { do_oop_nv(p); }
```

#### 参考(for your information): FilterOutOfRegionClosure::do_oop_nv()
See: [here](no3420hQK.html) for details
### 備考(Notes)
なお FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT の処理は
(おそらく FILTERINTOCSCLOSURE_DOHISTOGRAMCOUNT と同様のパフォーマンス上の理由で)
コンパイル時に on/off するようになっている (そして現状ではデフォルトは off).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp))
    #define FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT 0
```




### 詳細(Details)
See: [here](../doxygen/classFilterOutOfRegionClosure.html) for details

---
