---
layout: default
title: ConcurrentMark クラス関連のクラス (G1CMIsAliveClosure, CMBitMapRO, CMBitMap, CMMarkStack, CMRegionStack, ForceOverflowSettings, ConcurrentMark, CMTask, G1PrintRegionLivenessInfoClosure, 及びそれらの補助クラス(NoteStartOfMarkHRClosure, CMMarkRootsClosure, CMConcurrentMarkingTask, CalcLiveObjectsClosure, G1ParFinalCountTask, G1NoteEndOfConcMarkClosure, G1ParNoteEndTask, G1ParScrubRemSetTask, G1CMKeepAliveClosure, G1CMDrainMarkingStackClosure, G1CMParKeepAliveAndDrainClosure, G1CMParDrainMarkingStackClosure, G1RefProcTaskExecutor, G1RefProcTaskProxy, G1RefEnqueueTaskProxy, CMRemarkTask, PrintReachableOopClosure, PrintReachableObjectClosure, PrintReachableRegionClosure, CMGlobalObjectClosure, CSMarkOopClosure, CSMarkBitMapClosure, CompleteMarkingInCSHRClosure, ClearMarksInHRClosure, CMBitMapClosure, CMObjectClosure, CMOopClosure))
---
[Top](../index.html)

#### ConcurrentMark クラス関連のクラス (G1CMIsAliveClosure, CMBitMapRO, CMBitMap, CMMarkStack, CMRegionStack, ForceOverflowSettings, ConcurrentMark, CMTask, G1PrintRegionLivenessInfoClosure, 及びそれらの補助クラス(NoteStartOfMarkHRClosure, CMMarkRootsClosure, CMConcurrentMarkingTask, CalcLiveObjectsClosure, G1ParFinalCountTask, G1NoteEndOfConcMarkClosure, G1ParNoteEndTask, G1ParScrubRemSetTask, G1CMKeepAliveClosure, G1CMDrainMarkingStackClosure, G1CMParKeepAliveAndDrainClosure, G1CMParDrainMarkingStackClosure, G1RefProcTaskExecutor, G1RefProcTaskProxy, G1RefEnqueueTaskProxy, CMRemarkTask, PrintReachableOopClosure, PrintReachableObjectClosure, PrintReachableRegionClosure, CMGlobalObjectClosure, CSMarkOopClosure, CSMarkBitMapClosure, CompleteMarkingInCSHRClosure, ClearMarksInHRClosure, CMBitMapClosure, CMObjectClosure, CMOopClosure))

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Java プログラムの実行と並行(concurrent)に Marking 処理を行うクラス (See: [here](no2935d4w.html) for details).


### クラス一覧(class list)

  * [ConcurrentMark](#noDuaF_PP-)
  * [CMBitMapRO](#no2dukSzVC)
  * [CMBitMap](#noKc2PKQVT)
  * [CMMarkStack](#noMGXpBm1Q)
  * [CMRegionStack](#noW9foFfrO)
  * [ForceOverflowSettings](#noAx5bFmGZ)
  * [CMTask](#no3MwISmMS)
  * [G1CMIsAliveClosure](#noJmTfZIzF)
  * [G1PrintRegionLivenessInfoClosure](#noKhRUF7j7)
  * [NoteStartOfMarkHRClosure](#not5zoxsa-)
  * [CMMarkRootsClosure](#noIx5STqK8)
  * [CMConcurrentMarkingTask](#noua03K4gX)
  * [CalcLiveObjectsClosure](#nocfqhq8JX)
  * [G1ParFinalCountTask](#no-UGibR1a)
  * [G1NoteEndOfConcMarkClosure](#nocgSwULTX)
  * [G1ParNoteEndTask](#noU1uplKqz)
  * [G1ParScrubRemSetTask](#noM7KX_gxo)
  * [G1CMKeepAliveClosure](#no-ttxcD-y)
  * [G1CMDrainMarkingStackClosure](#noPxqwSVMC)
  * [G1CMParKeepAliveAndDrainClosure](#noDPFJml9e)
  * [G1CMParDrainMarkingStackClosure](#nomIRV-SqS)
  * [G1RefProcTaskExecutor](#notu7xNmRF)
  * [G1RefProcTaskProxy](#norPE2qI0u)
  * [G1RefEnqueueTaskProxy](#noDByTgXGD)
  * [CMRemarkTask](#nopYSRM3yH)
  * [PrintReachableOopClosure](#noEhvmhT-Z)
  * [PrintReachableObjectClosure](#noekh93_rV)
  * [PrintReachableRegionClosure](#no8FTxY2h4)
  * [CMGlobalObjectClosure](#nop2LOJe3P)
  * [CSMarkOopClosure](#noV9Xvfb-L)
  * [CSMarkBitMapClosure](#noahgKPZOc)
  * [CompleteMarkingInCSHRClosure](#noMvfZJ_sv)
  * [ClearMarksInHRClosure](#nolIA311Qp)
  * [CMBitMapClosure](#noV2tRdSUH)
  * [CMObjectClosure](#no1KBJWVhZ)
  * [CMOopClosure](#noHxFtA_hV)


---
## <a name="noDuaF_PP-" id="noDuaF_PP-">ConcurrentMark</a>

### 概要(Summary)
Concurrent Marking 処理を取りまとめるクラス. 以下のような役割を果たしている.

* Concurrent Marking 処理で使われるオブジェクト 
  (ConcurrentMarkThread, CMMarkStack, CMRegionStack, WorkGangBarrierSync) を管理し, 
  それらへのアクセサを提供する
* Concurrent Marking 処理用の定数／メソッドを提供する
  (ConcurrentMarkThread によるマルチスレッド処理時のスレッド数, etc)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    class ConcurrentMark: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _cm フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classConcurrentMark.html) for details

---
## <a name="no2dukSzVC" id="no2dukSzVC">CMBitMapRO</a>

### 概要(Summary)
Concurrent Marking の処理結果 (= どのオブジェクトが生きているかという情報) を記録するためのクラスの基底クラス
(論文中では "marking bitmaps").

内部的には (名前の通り) ビットマップになっており, 生きているオブジェクトに対応するビットが立てられる.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    // A generic CM bit map.  This is essentially a wrapper around the BitMap
    // class, with one bit per (1<<_shifter) HeapWords.
    
    class CMBitMapRO VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
実際の情報は _bm フィールドの BitMap オブジェクトに格納される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
      BitMap    _bm;               // the bit map itself
```

なお, このビットマップ上の 1bit は, 実際のヒープ空間上での "(1<<_shifter) HeapWords" に対応する.
_shifter フィールドの値はコンストラクタ引数によって決まるが, 
このクラス自体は abstract class なので, サブクラスである CMBitMap の生成箇所も参照.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
      const int _shifter;          // map to char or bit
```

#### 参考(for your information): CMBitMapRO::CMBitMapRO()
See: [here](no3420ahK.html) for details
### 備考(Notes)
名称の "RO" は "Read Only" の意味だと思われる. 
このクラスはビットマップを変更するメソッドを持たない (そういったメソッドはサブクラスの CMBitMap が持っている).

(このクラスは「immutable な」CMBitMap だとソースコード上で明示する役割があるのだと思われる.
ソースコード上でも, サブクラスの CMBitMap オブジェクトとして生成されたものがビットマップデータが完成した後は CMBitMapRO として使われていたりする)




### 詳細(Details)
See: [here](../doxygen/classCMBitMapRO.html) for details

---
## <a name="noKc2PKQVT" id="noKc2PKQVT">CMBitMap</a>

### 概要(Summary)
ConcurrentMark クラス内で使用される補助クラス.

CMBitMapRO クラスの具象サブクラス. このオブジェクト内に実際の Concurrent Marking 処理結果が格納される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    class CMBitMap : public CMBitMapRO {
```

### 使われ方(Usage)
以下の箇所に(のみ)格納されている.

* 各 ConcurrentMark オブジェクトの _markBitMap1 フィールド
* 各 ConcurrentMark オブジェクトの _markBitMap2 フィールド

### 備考(Notes)
どちらのフィールドのオブジェクトも, 
コンストラクタ引数として指定されている _shifter の値 (See: CMBitMapRO) は MinObjAlignment.

#### 参考(for your information): ConcurrentMark::ConcurrentMark()
See: [here](no3420nrQ.html) for details



### 詳細(Details)
See: [here](../doxygen/classCMBitMap.html) for details

---
## <a name="noMGXpBm1Q" id="noMGXpBm1Q">CMMarkStack</a>

### 概要(Summary)
ConcurrentMark クラス内で使用される補助クラス.
Gray なオブジェクト (= Concurrent Marking 処理で発見したが, まだ中までは調査できていないオブジェクト) を入れておくスタック
(論文中では "mark stack").


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    // Represents a marking stack used by the CM collector.
    // Ideally this should be GrowableArray<> just like MSC's marking stack(s).
    class CMMarkStack VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ConcurrentMark オブジェクトの _markStack フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(ConcurrentMark クラスの _markStack フィールドは, ポインタ型ではなく実体なので,
 ConcurrentMark オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
以下の箇所で使用されている (<= 他にも oops_do() 等の中で参照されている #TODO).

```
* ConcurrentMark オブジェクトの生成時に初期化される.

  ConcurrentMark::ConcurrentMark()
  -> ConcurrentMark::set_non_marking_state()
     -> ConcurrentMark::clear_marking_state()

* Concurrent Marking 処理中に, 見つかったオブジェクトがスタックに追加される

  ConcurrentMark::mark_stack_push(oop p)
  -> CMMarkStack::par_push()

* Concurrent Marking 処理中に, 見つかった配列オブジェクトがスタックに追加される

  ConcurrentMark::mark_stack_push(oop* arr, int n)
  -> CMMarkStack::par_push_arr()

* Concurrent Marking 処理中に, スタックに積まれた要素が取り出されて処理される
  
  CMTask::drain_global_stack()
  -> CMTask::get_entries_from_global_stack()
     -> ConcurrentMark::mark_stack_pop()
        -> CMMarkStack::par_pop_arr()

* Concurrent Marking 処理中に見つかった参照オブジェクトについても, 生きているものは参照先がスタックに積まれるので, 
  それらがスタックから取り出されて辿れる範囲全てが処理される

  ConcurrentMark::weakRefsWork()
  -> ReferenceProcessor::process_discovered_references()
     -> ...
        -> G1CMDrainMarkingStackClosure::do_void()
           -> CMMarkStack::drain()

* Concurrent Marking が終了した時点で, 初期状態にリセットされる

  ConcurrentMark::checkpointRootsFinal
  -> ConcurrentMark::set_non_marking_state()
     -> ConcurrentMark::clear_marking_state()
```




### 詳細(Details)
See: [here](../doxygen/classCMMarkStack.html) for details

---
## <a name="noW9foFfrO" id="noW9foFfrO">CMRegionStack</a>

### 概要(Summary)
Concurrent Marking 処理用の補助クラス.

Concurrent Marking 処理中に発生した Minor GC でコピー先として使用されたメモリ領域を記録しておくためのクラス
(Concurrent Mark 処理中に Minor GC が行われた場合, 
 コピー処理によってポインタが移動されることになるので, コピーされたオブジェクトについては再度調査しないといけない.
 このため, どこにコピーされたかを記録しておく必要がある).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    class CMRegionStack VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ConcurrentMark オブジェクトの _regionStack フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(ConcurrentMark クラスの _regionStack フィールドは, ポインタ型ではなく実体なので,
 ConcurrentMark オブジェクトの生成時に一緒に生成される)

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

```
* Minor GC 中にコピー先の HeapRegion が一杯になった時点で, そのコピー先がスタックに追加される

  G1ParCopyHelper::copy_to_survivor_space()
  -> G1ParScanThreadState::allocate()
     -> G1ParScanThreadState::allocate_slow()
        -> G1ParGCAllocBuffer::retire()
           -> GCLabBitMap::retire()
              -> ConcurrentMark::grayRegionIfNecessary()
                 -> ConcurrentMark::region_stack_push_lock_free()

* Minor GC で終わった時点で, 最後にコピー先として使用していた HeapRegion がスタックに追加される.

  G1ParEvacuateFollowersClosure::do_void()
  -> G1ParScanThreadState::retire_alloc_buffers()
     -> G1ParGCAllocBuffer::retire()
        -> (同上)

* Concurrent Marking 処理中に, スタックから要素が取り出され, その中のポインタが再調査される.

  CMTask::do_marking_step()
  -> CMTask::drain_region_stack()
     -> ConcurrentMark::region_stack_pop_lock_free()
```

なお, ConcurrentMark::_regionStack フィールドは以下の箇所でも参照されているが, 
これらの関数は使用箇所が見当たらない...

* ConcurrentMark::region_stack_push_with_lock()
* ConcurrentMark::region_stack_pop_with_lock()




### 詳細(Details)
See: [here](../doxygen/classCMRegionStack.html) for details

---
## <a name="noAx5bFmGZ" id="noAx5bFmGZ">ForceOverflowSettings</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時以外には空のクラスとして定義される).

強制的に ConcurrentMark 内のスタックが溢れた状態にするためのクラス
(スタックが溢れたとは, ConcurrentMark::has_overflown() が true を返す状態).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    class ForceOverflowSettings VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. ForceOverflowSettings::init() で初期化する

2. ForceOverflowSettings::should_force() を呼び出すと true が返される.
   一度呼び出すと次からは false が返されるが, 
   ForceOverflowSettings::update() を呼ぶとまた true を返す状態に戻すことが出来る.
   ただし, G1ConcMarkForceOverflow 回だけ ForceOverflowSettings::update() を呼んだ後は, 
   もう true の状態に戻すことは出来ない(= それ以降は false しか返されない).
   
   (なお, G1ConcMarkForceOverflow オプションの値が 0 の場合は, 
   1回目の ForceOverflowSettings::should_force() から false を返す.
   G1ConcMarkForceOverflow オプションのデフォルト値は 0.)

#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* 各 ConcurrentMark オブジェクトの _force_overflow_conc フィールド
* 各 ConcurrentMark オブジェクトの _force_overflow_stw フィールド




### 詳細(Details)
See: [here](../doxygen/classForceOverflowSettings.html) for details

---
## <a name="no3MwISmMS" id="no3MwISmMS">CMTask</a>

### 概要(Summary)
ConcurrentMark クラス内で使用される補助クラス.
実際のマーク処理を行う.

また Concurrent Marking 処理用の TerminatorTerminator クラスとしての役割もある
(マーク処理が完了したかどうかに応じて should_exit_termination() メソッドの返値が変わる (See: TerminatorTerminator)).

なお, CMTask オブジェクトは各 ConcurrentMarkThread に対して 1つ用意される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    // A class representing a marking task.
    class CMTask : public TerminatorTerminator {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
CMTask::do_marking_step() を呼ぶと, 
(SATBMarkQueueSet や CMMarkStack, CMRegionStack に入っているポインタを対象として) マーキング処理が行われる
(See: [here](no3420bUp.html) for details).

マーキング処理が全て終わったかどうかは CMTask::has_aborted() で確認できる.

#### インスタンスの格納場所(where its instances are stored)
各 ConcurrentMark オブジェクトの _tasks フィールドに(のみ)格納されている
(正確には, このフィールドは CMTask の配列を格納するフィールド.
この中に, 使用される全ての CMTask オブジェクトが格納されている).

#### 生成箇所(where its instances are created)
ConcurrentMark::ConcurrentMark() 内で(のみ)生成されている.
配列用のメモリ空間の確保もここで行う.

#### 削除箇所(where its instances are deleted)
ConcurrentMark::~ConcurrentMark() 内で(のみ)削除されている.




### 詳細(Details)
See: [here](../doxygen/classCMTask.html) for details

---
## <a name="noJmTfZIzF" id="noJmTfZIzF">G1CMIsAliveClosure</a>

### 概要(Summary)
G1CollectedHeap 使用時の Garbage Collection 処理で使用される補助クラス.

参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.
G1CMIsAliveClosure::do_object_b() メソッドが呼ばれると, 
(Concurrent Marking による mark 結果に基づいて)処理対象のオブジェクトが生きているかどうかを返す.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    // Closure used by CM during concurrent reference discovery
    // and reference processing (during remarking) to determine
    // if a particular object is alive. It is primarily used
    // to determine if referents of discovered reference objects
    // are alive. An instance is also embedded into the
    // reference processor as the _is_alive_non_header field
    class G1CMIsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
(このクラスは StackObj クラスだが, (原則に反して) 永続的に存在し続けるインスタンスが存在する).

各 G1CollectedHeap オブジェクトの _is_alive_closure フィールドに(のみ)格納されている.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* ReferenceProcessor::discover_reference() 
  
  (G1CollectedHeap::_is_alive_closure フィールドは, 
  同じく G1CollectedHeap の _ref_processor フィールドに格納されている 
  ReferenceProcessor のコンストラクタ引数として(のみ)使用されている.
  
  そしてその ReferenceProcessor オブジェクト内では
  ReferenceProcessor::discover_reference() で(のみ)使用されている.
  
  なお, この ReferenceProcessor オブジェクトは
  G1CollectedHeap の GC 処理 (Concurrent Marking 処理も含む) 中で参照オブジェクトの処理に使用されている) 
  (See: [here](no289169tf.html) for details)

* G1RefProcTaskProxy::work() 内 (局所変数として生成)
  
  (See: [here](no2935d4w.html) for details)

* ConcurrentMark::weakRefsWork() 内 (局所変数として生成)

  (See: [here](no2935d4w.html) for details)




### 詳細(Details)
See: [here](../doxygen/classG1CMIsAliveClosure.html) for details

---
## <a name="noKhRUF7j7" id="noKhRUF7j7">G1PrintRegionLivenessInfoClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) (See: G1PrintRegionLivenessInfo).

処理対象の HeapRegion の情報を指定された outputStream に出力する
(例えば, その HeapRegion 内の live オブジェクトの量, 等).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp))
    // Class that's used to to print out per-region liveness
    // information. It's currently used at the end of marking and also
    // after we sort the old regions at the end of the cleanup operation.
    class G1PrintRegionLivenessInfoClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* CollectionSetChooser::sortMarkedHeapRegions() 内
* ConcurrentMark::cleanup() 内




### 詳細(Details)
See: [here](../doxygen/classG1PrintRegionLivenessInfoClosure.html) for details

---
## <a name="not5zoxsa-" id="not5zoxsa-">NoteStartOfMarkHRClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Concurrent Marking 処理を補佐するためのクラス.
その Minor GC 後に Concurrent Marking 処理を開始させる場合に, 
各 HeapRegion の現在の top 位置を next TAMS に記録する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class NoteStartOfMarkHRClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
ConcurrentMark::checkpointRootsInitialPost() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classNoteStartOfMarkHRClosure.html) for details

---
## <a name="noIx5STqK8" id="noIx5STqK8">CMMarkRootsClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Initial Marking Pause 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

処理対象のオブジェクトにまだマークが付いていなければマークする
(= CMBitMap の該当箇所が立っていなければ, そのビットを立てる).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CMMarkRootsClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
ConcurrentMark::checkpointRootsInitial() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCMMarkRootsClosure.html) for details

---
## <a name="noua03K4gX" id="noua03K4gX">CMConcurrentMarkingTask</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Concurrent Marking 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

(SATBMarkQueueSet や CMMarkStack, CMRegionStack に入っているポインタに対して) マーキング処理を行う.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CMConcurrentMarkingTask: public AbstractGangTask {
```

### 使われ方(Usage)
ConcurrentMark::markFromRoots() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際のマーク処理は CMTask クラスに丸投げしている (See: CMTask).




### 詳細(Details)
See: [here](../doxygen/classCMConcurrentMarkingTask.html) for details

---
## <a name="nocfqhq8JX" id="nocfqhq8JX">CalcLiveObjectsClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Live Data Counting 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

処理対象の HeapRegion 内で生きているオブジェクトの量を計算する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CalcLiveObjectsClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ConcurrentMark::calcDesiredRegions() 内
* G1ParFinalCountTask::work() 内




### 詳細(Details)
See: [here](../doxygen/classCalcLiveObjectsClosure.html) for details

---
## <a name="no-UGibR1a" id="no-UGibR1a">G1ParFinalCountTask</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Live Data Counting 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

処理対象の HeapRegion 内で生きているオブジェクトの量を計算する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1ParFinalCountTask: public AbstractGangTask {
```

### 使われ方(Usage)
ConcurrentMark::cleanup() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際の計算処理は CalcLiveObjectsClosure に丸投げしている (See: CalcLiveObjectsClosure).




### 詳細(Details)
See: [here](../doxygen/classG1ParFinalCountTask.html) for details

---
## <a name="nocgSwULTX" id="nocgSwULTX">G1NoteEndOfConcMarkClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Cleanup 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

Concurrent Marking 処理の後片付けを行う. 具体的には以下の通り.

* next TAMS と prev TAMS を入れ替える.
* 生きているオブジェクトが存在しない HeapRegion を回収する.
* expand された SparsePRT オブジェクトの情報を収集する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1NoteEndOfConcMarkClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
G1ParNoteEndTask::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1NoteEndOfConcMarkClosure.html) for details

---
## <a name="noU1uplKqz" id="noU1uplKqz">G1ParNoteEndTask</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Cleanup 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

Concurrent Marking 処理の後片付けを行う. 具体的には以下の通り.

* next TAMS と prev TAMS を入れ替える.
* 生きているオブジェクトが存在しない HeapRegion を回収する.
* expand された SparsePRT オブジェクトの情報を収集する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1ParNoteEndTask: public AbstractGangTask {
```

### 使われ方(Usage)
ConcurrentMark::cleanup() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際の処理は G1NoteEndOfConcMarkClosure に丸投げしている (See: G1NoteEndOfConcMarkClosure).




### 詳細(Details)
See: [here](../doxygen/classG1ParNoteEndTask.html) for details

---
## <a name="noM7KX_gxo" id="noM7KX_gxo">G1ParScrubRemSetTask</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Cleanup 処理) で使用される補助クラス(StackObjクラス) (See: [here](no2935d4w.html) for details).

生きているオブジェクトがいない HeapRegion について, 
そこに付いている Remembered Set 用のデータ構造を解放する.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1ParScrubRemSetTask: public AbstractGangTask {
```

### 使われ方(Usage)
ConcurrentMark::cleanup() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1ParScrubRemSetTask.html) for details

---
## <a name="no-ttxcD-y" id="no-ttxcD-y">G1CMKeepAliveClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
まだマークが付いていないオブジェクトに対して,
マークを付け, そこから辿れるものを marking stack にプッシュする.

なお, このクラスは参照オブジェクトの処理をシングルスレッドで行う場合に使用される.
処理をマルチスレッドで行う場合は, 代わりに G1CMParKeepAliveAndDrainClosure が使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1CMKeepAliveClosure: public OopClosure {
```

### 使われ方(Usage)
ConcurrentMark::weakRefsWork() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1CMKeepAliveClosure.html) for details

---
## <a name="noPxqwSVMC" id="noPxqwSVMC">G1CMDrainMarkingStackClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
Concurrent Marking 処理中に見つかった参照オブジェクト(java.lang.ref オブジェクト)について, 
それらから指されている先を再帰的にマークする.

なお, このクラスは参照オブジェクトの処理をシングルスレッドで行う場合に使用される.
処理をマルチスレッドで行う場合は, 代わりに G1CMParDrainMarkingStackClosure が使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1CMDrainMarkingStackClosure: public VoidClosure {
```

### 使われ方(Usage)
ConcurrentMark::weakRefsWork() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1CMDrainMarkingStackClosure.html) for details

---
## <a name="noDPFJml9e" id="noDPFJml9e">G1CMParKeepAliveAndDrainClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
Concurrent Marking 処理中に見つかった参照オブジェクト(java.lang.ref オブジェクト)について, 
それらから指されている先を再帰的にマークする.

なお, このクラスは参照オブジェクトの処理をマルチスレッドで行う場合に使用される.
処理をシングルスレッドで行う場合は, 代わりに G1CMKeepAliveClosure が使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    // 'Keep Alive' closure used by parallel reference processing.
    // An instance of this closure is used in the parallel reference processing
    // code rather than an instance of G1CMKeepAliveClosure. We could have used
    // the G1CMKeepAliveClosure as it is MT-safe. Also reference objects are
    // placed on to discovered ref lists once so we can mark and push with no
    // need to check whether the object has already been marked. Using the
    // G1CMKeepAliveClosure would mean, however, having all the worker threads
    // operating on the global mark stack. This means that an individual
    // worker would be doing lock-free pushes while it processes its own
    // discovered ref list followed by drain call. If the discovered ref lists
    // are unbalanced then this could cause interference with the other
    // workers. Using a CMTask (and its embedded local data structures)
    // avoids that potential interference.
    class G1CMParKeepAliveAndDrainClosure: public OopClosure {
```

### 使われ方(Usage)
G1RefProcTaskProxy::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1CMParKeepAliveAndDrainClosure.html) for details

---
## <a name="nomIRV-SqS" id="nomIRV-SqS">G1CMParDrainMarkingStackClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
Concurrent Marking 処理中に見つかった参照オブジェクト(java.lang.ref オブジェクト)について, 
それらから指されている先を再帰的にマークする.

なお, このクラスは参照オブジェクトの処理をマルチスレッドで行う場合に使用される.
処理をシングルスレッドで行う場合は, 代わりに G1CMDrainMarkingStackClosure が使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1CMParDrainMarkingStackClosure: public VoidClosure {
```

### 使われ方(Usage)
G1RefProcTaskProxy::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1CMParDrainMarkingStackClosure.html) for details

---
## <a name="notu7xNmRF" id="notu7xNmRF">G1RefProcTaskExecutor</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

Concurrent Marking 処理で使用される AbstractRefProcTaskExecutor クラス
(つまり, Concurrent Marking 処理における参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するためのクラス
 (See: AbstractRefProcTaskExecutor)).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    // Implementation of AbstractRefProcTaskExecutor for G1
    class G1RefProcTaskExecutor: public AbstractRefProcTaskExecutor {
```

### 使われ方(Usage)
ConcurrentMark::weakRefsWork() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1RefProcTaskExecutor.html) for details

---
## <a name="norPE2qI0u" id="norPE2qI0u">G1RefProcTaskProxy</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された 
AbstractRefProcTaskExecutor::ProcessTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::ProcessTask))


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1RefProcTaskProxy: public AbstractGangTask {
```

### 使われ方(Usage)
G1RefProcTaskExecutor::execute() 内で(のみ)使用されている.

### 内部構造(Internal structure)
参照オブジェクト処理のマルチスレッド化自体は 
G1RefProcTaskExecutor クラスが (WorkGang を用いて) 行っている.

各 G1RefProcTaskProxy オブジェクトには
G1RefProcTaskExecutor クラスによって
実行すべき AbstractRefProcTaskExecutor::ProcessTask オブジェクトが 1つ割り当てられるので, 単にそれを実行するだけ.




### 詳細(Details)
See: [here](../doxygen/classG1RefProcTaskProxy.html) for details

---
## <a name="noDByTgXGD" id="noDByTgXGD">G1RefEnqueueTaskProxy</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された 
AbstractRefProcTaskExecutor::EnqueueTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::EnqueueTask))


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class G1RefEnqueueTaskProxy: public AbstractGangTask {
```

### 使われ方(Usage)
G1RefProcTaskExecutor::execute() 内で(のみ)使用されている.

### 内部構造(Internal structure)
参照オブジェクト処理のマルチスレッド化自体は 
G1RefProcTaskExecutor クラスが (WorkGang を用いて) 行っている.

各 G1RefProcTaskProxy オブジェクトには
G1RefProcTaskExecutor クラスによって
実行すべき AbstractRefProcTaskExecutor::EnqueueTask オブジェクトが 1つ割り当てられるので, 単にそれを実行するだけ.




### 詳細(Details)
See: [here](../doxygen/classG1RefEnqueueTaskProxy.html) for details

---
## <a name="nopYSRM3yH" id="nopYSRM3yH">CMRemarkTask</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Final Marking Pause 処理) で使用される補助クラス (See: [here](no2935d4w.html) for details).

Final Marking 処理を行う.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CMRemarkTask: public AbstractGangTask {
```

### 使われ方(Usage)
ConcurrentMark::checkpointRootsFinalWork() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCMRemarkTask.html) for details

---
## <a name="noEhvmhT-Z" id="noEhvmhT-Z">PrintReachableOopClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

PrintReachableObjectClosure クラス内で使用される補助クラス.
処理対象の oop に関する情報を出力する
(mark されているかどうか, アドレスが TAMS よりも前かどうか, 等).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class PrintReachableOopClosure: public OopClosure {
```

### 使われ方(Usage)
PrintReachableObjectClosure::do_object() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classPrintReachableOopClosure.html) for details

---
## <a name="noekh93_rV" id="noekh93_rV">PrintReachableObjectClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

PrintReachableRegionClosure クラス内で使用される補助クラス.
処理対象のオブジェクト, 及びそのオブジェクト内のポインタフィールド(oop)に関する情報を出力する
(mark されているかどうか, アドレスが TAMS よりも前かどうか, 等).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class PrintReachableObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
PrintReachableRegionClosure::doHeapRegion() 内で(のみ)使用されている.

### 内部構造(Internal structure)
オブジェクト内のポインタフィールドの処理については, 
PrintReachableOopClosure に丸投げしている (See: PrintReachableOopClosure).




### 詳細(Details)
See: [here](../doxygen/classPrintReachableObjectClosure.html) for details

---
## <a name="no8FTxY2h4" id="no8FTxY2h4">PrintReachableRegionClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

処理対象の HeapRegion 内にある全てのオブジェクトの情報を出力する
(mark されているかどうか, アドレスが TAMS よりも前かどうか, 等).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class PrintReachableRegionClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
ConcurrentMark::print_reachable() 内で(のみ)使用されている.

### 内部構造(Internal structure)
実際の出力処理は PrintReachableObjectClosure に丸投げしている (See: PrintReachableObjectClosure).




### 詳細(Details)
See: [here](../doxygen/classPrintReachableRegionClosure.html) for details

---
## <a name="nop2LOJe3P" id="nop2LOJe3P">CMGlobalObjectClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC 時に, 残っている Concurrent Marking 処理を完了させるための Closure クラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CMGlobalObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
ConcurrentMark::drainAllSATBBuffers() 内で(のみ)使用されている (See: [here](no2935YzN.html) for details).




### 詳細(Details)
See: [here](../doxygen/classCMGlobalObjectClosure.html) for details

---
## <a name="noV9Xvfb-L" id="noV9Xvfb-L">CSMarkOopClosure</a>

### 概要(Summary)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CSMarkOopClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CSMarkBitMapClosure オブジェクトの _oop_cl フィールドに(のみ)格納されている.

#### 使用箇所(where its instances are used)
CSMarkBitMapClosure::do_bit() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCSMarkOopClosure.html) for details

---
## <a name="noahgKPZOc" id="noahgKPZOc">CSMarkBitMapClosure</a>

### 概要(Summary)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CSMarkBitMapClosure: public BitMapClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompleteMarkingInCSHRClosure オブジェクトの _bit_cl フィールドに(のみ)格納されている.

#### 使用箇所(where its instances are used)
CompleteMarkingInCSHRClosure::doHeapRegion() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCSMarkBitMapClosure.html) for details

---
## <a name="noMvfZJ_sv" id="noMvfZJ_sv">CompleteMarkingInCSHRClosure</a>

### 概要(Summary)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class CompleteMarkingInCSHRClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
ConcurrentMark::complete_marking_in_collection_set() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
(略) (See: [here](no2935YzN.html) for details)
-> G1CollectedHeap::evacuate_collection_set()
   -> ConcurrentMark::complete_marking_in_collection_set()
```




### 詳細(Details)
See: [here](../doxygen/classCompleteMarkingInCSHRClosure.html) for details

---
## <a name="nolIA311Qp" id="nolIA311Qp">ClearMarksInHRClosure</a>

### 概要(Summary)

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    class ClearMarksInHRClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
ConcurrentMark::complete_marking_in_collection_set() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classClearMarksInHRClosure.html) for details

---
## <a name="noV2tRdSUH" id="noV2tRdSUH">CMBitMapClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Concurrent Marking 処理と Final Marking Pause 処理) で使用される補助クラス 
(See: [here](no2935d4w.html) for details).

マークされているオブジェクト (= CMBitMap の該当箇所が立っているオブジェクト) から再帰的に辿れる範囲をマークするための Closure クラス.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    // Closure for iteration over bitmaps
    class CMBitMapClosure : public BitMapClosure {
```

### 使われ方(Usage)
CMTask::do_marking_step() 内で(のみ)使用されている (See: [here](no3420bUp.html) for details).




### 詳細(Details)
See: [here](../doxygen/classCMBitMapClosure.html) for details

---
## <a name="no1KBJWVhZ" id="no1KBJWVhZ">CMObjectClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Concurrent Marking 処理と Final Marking Pause 処理) で使用される補助クラス 
(See: [here](no2935d4w.html) for details).

処理対象のオブジェクトにマークを付ける (= CMBitMap の該当箇所のビットを立てる).
また, 既に処理を終えた HeapRegion 内を指している場合には, 再調査の必要があるので, local queue への登録も行う.

なお, このクラスは ObjectClosure (つまり, 処理対象は oop).
処理対象が oop* の場合には CMOopClosure が使用される.

また, このクラスは現状では SATB buffer の処理にしか使用されていない
(SATB 方式の write barrier で記録された(変更前の)ポインタフィールドの値に対して, その差し先にマークを付ける処理).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    // Closure for iterating over objects, currently only used for
    // processing SATB buffers.
    class CMObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
CMTask::drain_satb_buffers() 内で(のみ)使用されている (See: [here](no3420bUp.html) for details).




### 詳細(Details)
See: [here](../doxygen/classCMObjectClosure.html) for details

---
## <a name="noHxFtA_hV" id="noHxFtA_hV">CMOopClosure</a>

### 概要(Summary)
G1GC の ConcurrentMarkThread (の Concurrent Marking 処理と Final Marking Pause 処理) で使用される補助クラス 
(See: [here](no2935d4w.html) for details).

処理対象のオブジェクトにマークを付ける (= CMBitMap の該当箇所のビットを立てる).
また, 既に処理を終えた HeapRegion 内を指している場合には, 再調査の必要があるので, local queue への登録も行う.

なお, このクラスは OopClosure (つまり, 処理対象は oop*).
処理対象が oop の場合には CMObjectClosure が使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp))
    // Closure for iterating over object fields
    class CMOopClosure : public OopClosure {
```

### 使われ方(Usage)
CMTask::do_marking_step() 内で(のみ)使用されている 
(より正確に言うと, そこから呼び出される CMTask::scan_object() 内で(のみ)使用されている)
(See: [here](no3420bUp.html) for details).




### 詳細(Details)
See: [here](../doxygen/classCMOopClosure.html) for details

---
