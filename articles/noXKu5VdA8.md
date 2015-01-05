---
layout: default
title: G1CollectedHeap クラス関連のクラス (YoungList, MutatorAllocRegion, G1CollectedHeap, GCLabBitMapClosure, GCLabBitMap, G1ParGCAllocBuffer, G1ParScanThreadState, 及びそれらの補助クラス(RefineCardTableEntryClosure, ClearLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryFastClosure, PostMCRemSetClearClosure, PostMCRemSetInvalidateClosure, RebuildRSOutOfRegionClosure, ParRebuildRSTask, SumUsedClosure, SumUsedRegionsClosure, IterateOopClosureRegionClosure, IterateObjectClosureRegionClosure, SpaceClosureRegionClosure, ResetClaimValuesClosure, CheckClaimValuesClosure, VerifyLivenessOopClosure, VerifyObjsInRegionClosure, PrintObjsInRegionClosure, VerifyRegionClosure, VerifyRootsClosure, G1ParVerifyTask, PrintRegionClosure, VerifyMarkedObjsClosure, FindGCAllocRegion, G1IsAliveClosure, G1KeepAliveClosure, UpdateRSetDeferred, RemoveSelfPointerClosure, G1ParEvacuateFollowersClosure, G1ParTask, SaveMarksClosure, G1ParCleanupCTTask, G1VerifyCardTableCleanup, NoYoungRegionsClosure, RegionResetter, VerifyRegionListsClosure))
---
[Top](../index.html)

#### G1CollectedHeap クラス関連のクラス (YoungList, MutatorAllocRegion, G1CollectedHeap, GCLabBitMapClosure, GCLabBitMap, G1ParGCAllocBuffer, G1ParScanThreadState, 及びそれらの補助クラス(RefineCardTableEntryClosure, ClearLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryClosure, RedirtyLoggedCardTableEntryFastClosure, PostMCRemSetClearClosure, PostMCRemSetInvalidateClosure, RebuildRSOutOfRegionClosure, ParRebuildRSTask, SumUsedClosure, SumUsedRegionsClosure, IterateOopClosureRegionClosure, IterateObjectClosureRegionClosure, SpaceClosureRegionClosure, ResetClaimValuesClosure, CheckClaimValuesClosure, VerifyLivenessOopClosure, VerifyObjsInRegionClosure, PrintObjsInRegionClosure, VerifyRegionClosure, VerifyRootsClosure, G1ParVerifyTask, PrintRegionClosure, VerifyMarkedObjsClosure, FindGCAllocRegion, G1IsAliveClosure, G1KeepAliveClosure, UpdateRSetDeferred, RemoveSelfPointerClosure, G1ParEvacuateFollowersClosure, G1ParTask, SaveMarksClosure, G1ParCleanupCTTask, G1VerifyCardTableCleanup, NoYoungRegionsClosure, RegionResetter, VerifyRegionListsClosure))

これらは, Java ヒープ領域を管理するためのクラス (See: [here](no3718kvd.html) for details).

Java ヒープ領域を管理するクラスは使用する GC アルゴリズムによって異なるが,
これらのクラスは GC アルゴリズムが G1GC の場合に使用される.
(See: GenCollectedHeap, ParallelScavengeHeap)


### クラス一覧(class list)

  * [G1CollectedHeap](#no1PIInDpx)
  * [YoungList](#noAKKAiFLw)
  * [MutatorAllocRegion](#nofZODHEJU)
  * [G1ParGCAllocBuffer](#noeHjDhPzv)
  * [GCLabBitMap](#nozlyPENlO)
  * [GCLabBitMapClosure](#noZZGgVMpl)
  * [G1ParScanThreadState](#noiM3effas)
  * [RefineCardTableEntryClosure](#nogHq0Wcou)
  * [ClearLoggedCardTableEntryClosure](#no7wizIg5q)
  * [RedirtyLoggedCardTableEntryClosure](#nohhmu5GYl)
  * [RedirtyLoggedCardTableEntryFastClosure](#no-AtOaDBH)
  * [PostMCRemSetClearClosure](#norbF5B5OB)
  * [PostMCRemSetInvalidateClosure](#notWeTfZwX)
  * [RebuildRSOutOfRegionClosure](#nowMe5m9Kf)
  * [ParRebuildRSTask](#nonQ1GYhAL)
  * [SumUsedClosure](#noIzrF_VeT)
  * [SumUsedRegionsClosure](#noskWJekhr)
  * [IterateOopClosureRegionClosure](#no0xevAxFK)
  * [IterateObjectClosureRegionClosure](#noBN-8lXvc)
  * [SpaceClosureRegionClosure](#noSk6aOeAU)
  * [ResetClaimValuesClosure](#noOA3rQgUA)
  * [CheckClaimValuesClosure](#nokp-dl-4l)
  * [VerifyLivenessOopClosure](#nobpMESrkQ)
  * [VerifyObjsInRegionClosure](#nor10dvm86)
  * [PrintObjsInRegionClosure](#nohCs-vjJ9)
  * [VerifyRegionClosure](#noVYTRoIgH)
  * [VerifyRootsClosure](#nohAXIM_4C)
  * [G1ParVerifyTask](#noqdtTpUGi)
  * [PrintRegionClosure](#noc75HHpuv)
  * [VerifyMarkedObjsClosure](#noNpgTyG8w)
  * [FindGCAllocRegion](#no7djYUaU4)
  * [G1IsAliveClosure](#no0FiI45cN)
  * [G1KeepAliveClosure](#nom5QrhIVU)
  * [UpdateRSetDeferred](#noUOTjl0_A)
  * [RemoveSelfPointerClosure](#nomBxVH4Ra)
  * [G1ParEvacuateFollowersClosure](#noWsif6LhY)
  * [G1ParTask](#nopbvNS8Go)
  * [SaveMarksClosure](#nodN44Ql2C)
  * [G1ParCleanupCTTask](#no9FY64-0W)
  * [G1VerifyCardTableCleanup](#no3FzkHj0B)
  * [NoYoungRegionsClosure](#noK5HkRWLF)
  * [RegionResetter](#nocLWd0QzX)
  * [VerifyRegionListsClosure](#nowrBBrH_c)


---
## <a name="no1PIInDpx" id="no1PIInDpx">G1CollectedHeap</a>

### 概要(Summary)
Java ヒープ領域の管理を担当するクラス(CollectedHeapクラス)の1つ (See: [here](no3718kvd.html) for details).

このクラスは, GC アルゴリズムが G1GC の場合用.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class G1CollectedHeap : public SharedHeap {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* VirtualSpace 	_g1_storage
* MemRegion 	_g1_reserved
* MemRegion 	_g1_committed
* MemRegion 	_g1_max_committed
* MasterFreeRegionList 	_free_list
* SecondaryFreeRegionList 	_secondary_free_list
* MasterHumongousRegionSet 	_humongous_set
* size_t 	_expansion_regions
* G1BlockOffsetSharedArray * 	_bot_shared
* HeapRegionSeq * 	_hrs
* MutatorAllocRegion 	_mutator_alloc_region
* HeapRegion * 	_gc_alloc_regions [GCAllocPurposeCount]
* size_t 	_gc_alloc_region_counts [GCAllocPurposeCount]
* HeapRegion * 	_retained_gc_alloc_regions [GCAllocPurposeCount]
* bool 	_retain_gc_alloc_region [GCAllocPurposeCount]
* HeapRegion * 	_gc_alloc_region_list
* G1MonitoringSupport * 	_g1mm
* size_t 	_summary_bytes_used
* bool * 	_in_cset_fast_test
* bool * 	_in_cset_fast_test_base
* size_t 	_in_cset_fast_test_length
* volatile unsigned 	_gc_time_stamp
* size_t * 	_surviving_young_words
* volatile unsigned int 	_full_collections_completed

* int 	_num_humongous_regions
* YoungList * 	_young_list
* G1CollectorPolicy * 	_g1_policy
* G1RemSet * 	_g1_rem_set
* ModRefBarrierSet * 	_mr_bs
* DirtyCardQueueSet 	_dirty_card_queue_set
* HeapRegionRemSetIterator ** 	_rem_set_iterator
* RefineCardTableEntryClosure * 	_refine_cte_cl
* DirtyCardQueueSet 	_into_cset_dirty_card_queue_set
* ConcurrentMark * 	_cm
* ConcurrentMarkThread * 	_cmThread

* bool 	_mark_in_progress

  Concurrent Marking を実行中かどうかを示す.
  以下のメソッドがアクセサ.

    * G1CollectedHeap::set_marking_complete()
    * G1CollectedHeap::set_marking_started()
    * G1CollectedHeap::mark_in_progress()

* ConcurrentG1Refine * 	_cg1r
* RefToScanQueueSet * 	_task_queues

* bool 	_evacuation_failed
  
  Minor GC が失敗した際に true になる
  (このフィールドが true だと, Minor GC 後に失敗した GC の後始末が行われる (See: [here](no3420RUs.html) for details)).

* GrowableArray< oop > * 	_objs_with_preserved_marks
* GrowableArray< markOop > * 	_preserved_marks_of_objs
* GrowableArray< oop > * 	_evac_failure_scan_stack
* OopsInHeapRegionClosure * 	_evac_failure_closure
* bool 	_drain_in_progress
  
  Minor GC が失敗した場合の処理で(のみ)使用されるフィールド.
  G1CollectedHeap::handle_evacuation_failure_common() の呼び出しが再帰にならないようにするためのフラグ
  (See: [here](no3420RUs.html) for details).

* G1CMIsAliveClosure 	_is_alive_closure
* ReferenceProcessor * 	_ref_processor
* SubTasksDone * 	_process_strong_tasks
* volatile bool 	_free_regions_coming
* bool 	_full_collection
* HeapRegion * 	_dirty_cards_region_list
  
  Minor GC 処理中に使用されるフィールド.
  Collection Set に含まれた HeapRegion の中で Barrier Set をクリアする必要があるものが格納される
  (See: G1CollectedHeap::push_dirty_cards_region(), G1CollectedHeap::pop_dirty_cards_region()).

* size_t 	_max_heap_capacity



```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // Storage for the G1 heap (excludes the permanent generation).
      VirtualSpace _g1_storage;
      MemRegion    _g1_reserved;
    
      // The part of _g1_storage that is currently committed.
      MemRegion _g1_committed;
    
      // The maximum part of _g1_storage that has ever been committed.
      MemRegion _g1_max_committed;
    
      // The master free list. It will satisfy all new region allocations.
      MasterFreeRegionList      _free_list;
    
      // The secondary free list which contains regions that have been
      // freed up during the cleanup process. This will be appended to the
      // master free list when appropriate.
      SecondaryFreeRegionList   _secondary_free_list;
    
      // It keeps track of the humongous regions.
      MasterHumongousRegionSet  _humongous_set;
    
      // The number of regions we could create by expansion.
      size_t _expansion_regions;
    
      // The block offset table for the G1 heap.
      G1BlockOffsetSharedArray* _bot_shared;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The sequence of all heap regions in the heap.
      HeapRegionSeq* _hrs;
    
      // Alloc region used to satisfy mutator allocation requests.
      MutatorAllocRegion _mutator_alloc_region;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The to-space memory regions into which objects are being copied during
      // a GC.
      HeapRegion* _gc_alloc_regions[GCAllocPurposeCount];
      size_t _gc_alloc_region_counts[GCAllocPurposeCount];
      // These are the regions, one per GCAllocPurpose, that are half-full
      // at the end of a collection and that we want to reuse during the
      // next collection.
      HeapRegion* _retained_gc_alloc_regions[GCAllocPurposeCount];
      // This specifies whether we will keep the last half-full region at
      // the end of a collection so that it can be reused during the next
      // collection (this is specified per GCAllocPurpose)
      bool _retain_gc_alloc_region[GCAllocPurposeCount];
    
      // A list of the regions that have been set to be alloc regions in the
      // current collection.
      HeapRegion* _gc_alloc_region_list;
    
      // Helper for monitoring and management support.
      G1MonitoringSupport* _g1mm;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // Outside of GC pauses, the number of bytes used in all regions other
      // than the current allocation region.
      size_t _summary_bytes_used;
    
      // This is used for a quick test on whether a reference points into
      // the collection set or not. Basically, we have an array, with one
      // byte per region, and that byte denotes whether the corresponding
      // region is in the collection set or not. The entry corresponding
      // the bottom of the heap, i.e., region 0, is pointed to by
      // _in_cset_fast_test_base.  The _in_cset_fast_test field has been
      // biased so that it actually points to address 0 of the address
      // space, to make the test as fast as possible (we can simply shift
      // the address to address into it, instead of having to subtract the
      // bottom of the heap from the address before shifting it; basically
      // it works in the same way the card table works).
      bool* _in_cset_fast_test;
    
      // The allocated array used for the fast test on whether a reference
      // points into the collection set or not. This field is also used to
      // free the array.
      bool* _in_cset_fast_test_base;
    
      // The length of the _in_cset_fast_test_base array.
      size_t _in_cset_fast_test_length;
    
      volatile unsigned _gc_time_stamp;
    
      size_t* _surviving_young_words;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // Keeps track of how many "full collections" (i.e., Full GCs or
      // concurrent cycles) we have completed. The number of them we have
      // started is maintained in _total_full_collections in CollectedHeap.
      volatile unsigned int _full_collections_completed;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The number of regions allocated to hold humongous objects.
      int         _num_humongous_regions;
      YoungList*  _young_list;
    
      // The current policy object for the collector.
      G1CollectorPolicy* _g1_policy;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The g1 remembered set of the heap.
      G1RemSet* _g1_rem_set;
      // And it's mod ref barrier set, used to track updates for the above.
      ModRefBarrierSet* _mr_bs;
    
      // A set of cards that cover the objects for which the Rsets should be updated
      // concurrently after the collection.
      DirtyCardQueueSet _dirty_card_queue_set;
    
      // The Heap Region Rem Set Iterator.
      HeapRegionRemSetIterator** _rem_set_iterator;
    
      // The closure used to refine a single card.
      RefineCardTableEntryClosure* _refine_cte_cl;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // A DirtyCardQueueSet that is used to hold cards that contain
      // references into the current collection set. This is used to
      // update the remembered sets of the regions in the collection
      // set in the event of an evacuation failure.
      DirtyCardQueueSet _into_cset_dirty_card_queue_set;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The concurrent marker (and the thread it runs in.)
      ConcurrentMark* _cm;
      ConcurrentMarkThread* _cmThread;
      bool _mark_in_progress;
    
      // The concurrent refiner.
      ConcurrentG1Refine* _cg1r;
    
      // The parallel task queues
      RefToScanQueueSet *_task_queues;
    
      // True iff a evacuation has failed in the current collection.
      bool _evacuation_failed;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // When one is non-null, so is the other.  Together, they each pair is
      // an object with a preserved mark, and its mark value.
      GrowableArray<oop>*     _objs_with_preserved_marks;
      GrowableArray<markOop>* _preserved_marks_of_objs;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The stack of evac-failure objects left to be scanned.
      GrowableArray<oop>*    _evac_failure_scan_stack;
      // The closure to apply to evac-failure objects.
    
      OopsInHeapRegionClosure* _evac_failure_closure;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // True iff an invocation of "drain_scan_stack" is in progress; to
      // prevent unnecessary recursion.
      bool _drain_in_progress;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // Instance of the concurrent mark is_alive closure for embedding
      // into the reference processor as the is_alive_non_header. This
      // prevents unnecessary additions to the discovered lists during
      // concurrent discovery.
      G1CMIsAliveClosure _is_alive_closure;
    
      // ("Weak") Reference processing support
      ReferenceProcessor* _ref_processor;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      SubTasksDone* _process_strong_tasks;
    
      volatile bool _free_regions_coming;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The following is just to alert the verification code
      // that a full collection has occurred and that the
      // remembered sets are no longer up to date.
      bool _full_collection;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      // The dirty cards region list is used to record a subset of regions
      // whose cards need clearing. The list if populated during the
      // remembered set scanning and drained during the card table
      // cleanup. Although the methods are reentrant, population/draining
      // phases must not overlap. For synchronization purposes the last
      // element on the list points to itself.
      HeapRegion* _dirty_cards_region_list;
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
      size_t _max_heap_capacity;
```




### 詳細(Details)
See: [here](../doxygen/classG1CollectedHeap.html) for details

---
## <a name="noAKKAiFLw" id="noAKKAiFLw">YoungList</a>

### 概要(Summary)
G1CollectedHeap クラス用の補助クラス.

オブジェクトの確保用に最近使用された HeapRegion を記録しておくためのコンテナクラス(線形リスト).
この中に格納されている HeapRegion が世代別 GC の New 領域 (Eden および Survivor 領域) に相当する .
このリストがある程度長くなると Minor GC 処理が開始される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class YoungList : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _young_list フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectedHeap::G1CollectedHeap() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
以下のパスで(のみ) HeapRegion が追加されている.

<div class="flow-abst"><pre>
G1CollectedHeap::attempt_allocation()
-&gt; G1CollectedHeap::attempt_allocation_slow()
   -&gt; G1AllocRegion::attempt_allocation_locked()
      -&gt; G1AllocRegion::new_alloc_region_and_allocate()
         -&gt; MutatorAllocRegion::allocate_new_region()
            -&gt; G1CollectedHeap::new_mutator_alloc_region()
               -&gt; G1CollectedHeap::set_region_short_lived_locked()
                  -&gt; YoungList::push_region()
   -&gt; G1AllocRegion::attempt_allocation_force()
      -&gt; G1AllocRegion::new_alloc_region_and_allocate()
         -&gt; (同上)

...
-&gt; G1CollectedHeap::attempt_allocation_at_safepoint()
   -&gt; G1AllocRegion::attempt_allocation_locked()
      -&gt; (同上)
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classYoungList.html) for details

---
## <a name="nofZODHEJU" id="nofZODHEJU">MutatorAllocRegion</a>

### 概要(Summary)
G1AllocRegion クラスの具象サブクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class MutatorAllocRegion : public G1AllocRegion {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _mutator_alloc_region フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(G1CollectedHeap クラスの _mutator_alloc_region フィールドは, ポインタ型ではなく実体なので,
 G1CollectedHeap オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classMutatorAllocRegion.html) for details

---
## <a name="noeHjDhPzv" id="noeHjDhPzv">G1ParGCAllocBuffer</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される補助クラス (See: [here](no2935YzN.html) for details).

Minor GC 中に各 GC Thread (GangWorker) がコピー先として使用する To 領域や Old 領域を管理するクラス
(論文中では "GCLABs(GC thread local allocation buffers)").


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class G1ParGCAllocBuffer: public ParGCAllocBuffer {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている

* 各 G1ParScanThreadState オブジェクトの _surviving_alloc_buffer フィールド
  
  To 領域用の HeapRegin を管理する

* 各 G1ParScanThreadState オブジェクトの _tenured_alloc_buffer フィールド

  Old 領域用の HeapRegin を管理する

#### 生成箇所(where its instances are created)
(なお, このクラスは CHeapObj クラスだが, G1ParScanThreadState クラス(StackObjクラス)のフィールドとしてのみ生成されている)

* (G1ParScanThreadState クラスの _surviving_alloc_buffer フィールドは, ポインタ型ではなく実体なので,
  G1ParScanThreadState オブジェクトの生成時に一緒に生成される)

* (G1ParScanThreadState クラスの _tenured_alloc_buffer フィールドは, ポインタ型ではなく実体なので,
  G1ParScanThreadState オブジェクトの生成時に一緒に生成される)

### 備考(Notes)
なお, スーパークラスの ParGCAllocBuffer は parNew/ 以下で定義されている模様 
(何か妙な気もするが, 必要ないのにわざわざ新しいクラスを定義するのも無駄がある?? #TODO).




### 詳細(Details)
See: [here](../doxygen/classG1ParGCAllocBuffer.html) for details

---
## <a name="nozlyPENlO" id="nozlyPENlO">GCLabBitMap</a>

### 概要(Summary)
G1ParGCAllocBuffer クラス内で使用される補助クラス 
(つまり, G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される補助クラス (See: [here](no2935YzN.html) for details)).

G1ParGCAllocBuffer がコピー先として使用している HeapRegion 内のオブジェクトについて, 
どのオブジェクトが生きているかという情報を記録していくためのビットマップ.
作業中に一時的に情報がここに蓄えられ, 適当な契機で ConcurrentMark 内にある mark bitmap (CMBitMap) に反映される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class GCLabBitMap: public BitMap {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. GCLabBitMap::GCLabBitMap() 又は GCLabBitMap::set_buffer() で, ターゲットとなる HeapRegion を設定する.
2. GCLabBitMap::mark() で, 指定箇所のオブジェクトが live だと記録する (= ビットを立てる).
3. GCLabBitMap::retire() で, 記録結果を ConcurrentMark 内の mark bitmap (CMBitMap) に反映する.

#### インスタンスの格納場所(where its instances are stored)
各 G1ParGCAllocBuffer オブジェクトの _bitmap フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
(G1ParGCAllocBuffer クラスの _bitmap フィールドは, ポインタ型ではなく実体なので,
 G1ParGCAllocBuffer オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classGCLabBitMap.html) for details

---
## <a name="noZZGgVMpl" id="noZZGgVMpl">GCLabBitMapClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

ConcurrentMark オブジェクト内の mark bitmap が正しいかどうかのチェックを行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    #ifndef PRODUCT
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class GCLabBitMapClosure: public BitMapClosure {
```

### 使われ方(Usage)
GCLabBitMap::verify() 内で(のみ)使用されている.

### 内部構造(Internal structure)
GCLabBitMapClosure::do_bit() の内部は, 単に guarantee() によるチェックを行うだけ.




### 詳細(Details)
See: [here](../doxygen/classGCLabBitMapClosure.html) for details

---
## <a name="noiM3effas" id="noiM3effas">G1ParScanThreadState</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される補助クラス(StackObjクラス) (See: [here](no2935YzN.html) for details).

処理を行う GC Thread (WorkGang) の状態を管理するためのクラス. 以下のような情報を格納している.

* 調査中のポインタを積んでいくキュー(RefToScanQueue) (See: G1ParPushHeapRSClosure::do_oop_nv()))

* コピー先として使用する HeapRegion (を示す G1ParGCAllocBuffer)

* 昇格のための閾値(tenuring threshould)を計算するための ageTable

* Minor GC 処理で使用する Closure オブジェクト 
  (OopsInHeapRegionClosure, G1ParScanHeapEvacClosure, G1ParScanPartialArrayClosure)
 

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.hpp))
    class G1ParScanThreadState : public StackObj {
```

### 使われ方(Usage)
G1ParTask::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1ParScanThreadState.html) for details

---
## <a name="nogHq0Wcou" id="nogHq0Wcou">RefineCardTableEntryClosure</a>

### 概要(Summary)
G1CollectedHeap の Remembered Set の修正処理(= ConcurrentG1RefineThread の処理)で使用される補助クラス
(See: [here](no2935dGZ.html) for details).

DirtyCardQueue 内の各 card に対して, 実際に Remembered Set の修正処理を行う Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RefineCardTableEntryClosure: public CardTableEntryClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 G1CollectedHeap オブジェクトの _refine_cte_cl フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
G1CollectedHeap::initialize() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
JavaThread::dirty_card_queue_set() に格納されている DirtyCardQueueSet オブジェクトの中で(のみ)使用されている
(より正確に言うと, G1CollectedHeap::_refine_cte_cl フィールドは, 
 この DirtyCardQueueSet オブジェクトを対象とした
 DirtyCardQueueSet::set_closure() の引数としてのみ参照される).




### 詳細(Details)
See: [here](../doxygen/classRefineCardTableEntryClosure.html) for details

---
## <a name="no7wizIg5q" id="no7wizIg5q">ClearLoggedCardTableEntryClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class ClearLoggedCardTableEntryClosure: public CardTableEntryClosure {
```

### 使われ方(Usage)
G1CollectedHeap::check_ct_logs_at_safepoint() 内で(のみ)使用されている
(が, この関数自体が使われていないような...?? #TODO)




### 詳細(Details)
See: [here](../doxygen/classClearLoggedCardTableEntryClosure.html) for details

---
## <a name="nohhmu5GYl" id="nohhmu5GYl">RedirtyLoggedCardTableEntryClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RedirtyLoggedCardTableEntryClosure: public CardTableEntryClosure {
```

### 使われ方(Usage)
G1CollectedHeap::check_ct_logs_at_safepoint() 内で(のみ)使用されている
(が, この関数自体が使われていないような...?? #TODO)




### 詳細(Details)
See: [here](../doxygen/classRedirtyLoggedCardTableEntryClosure.html) for details

---
## <a name="no-AtOaDBH" id="no-AtOaDBH">RedirtyLoggedCardTableEntryFastClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

G1CollectedHeap::dirty_card_queue_set() 内の各 card について (= 処理が遅延されていた各 card について), 
card の dirty 化を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RedirtyLoggedCardTableEntryFastClosure : public CardTableEntryClosure {
```

### 使われ方(Usage)
G1CollectedHeap::evacuate_collection_set() 内で(のみ)使用されている

### 備考(Notes)
develop オプションである G1DeferredRSUpdate を false に変更した場合, このクラスは使用されなくなる.




### 詳細(Details)
See: [here](../doxygen/classRedirtyLoggedCardTableEntryFastClosure.html) for details

---
## <a name="norbF5B5OB" id="norbF5B5OB">PostMCRemSetClearClosure</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される Closure クラス (See: [here](no2935ATn.html) for details).

(Major GC の処理後に) 全ての HeapRegion の Remembered Set 情報をリセットするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class PostMCRemSetClearClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::do_collection() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classPostMCRemSetClearClosure.html) for details

---
## <a name="notWeTfZwX" id="notWeTfZwX">PostMCRemSetInvalidateClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class PostMCRemSetInvalidateClosure: public HeapRegionClosure {
```




### 詳細(Details)
See: [here](../doxygen/classPostMCRemSetInvalidateClosure.html) for details

---
## <a name="nowMe5m9Kf" id="nowMe5m9Kf">RebuildRSOutOfRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される補助クラス (See: [here](no2935ATn.html) for details).

(Major GC の処理後に) 全ての HeapRegion の Remembered Set 情報を作成し直すための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RebuildRSOutOfRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::do_collection() 内で(のみ)使用されている
(より正確には, G1CollectedHeap::do_collection() 内と,
そこから呼び出される ParRebuildRSTask::work() 内で使用される補助クラス).

### 内部構造(Internal structure)
実際の処理のほとんどは UpdateRSOopClosure に丸投げしている
(より正確に言うと, RebuildRSOutOfRegionClosure が各 HeapRegion に対して呼び出され, 
その中で HeapRegion 内の各 oop に対して UpdateRSOopClosure が呼び出される).




### 詳細(Details)
See: [here](../doxygen/classRebuildRSOutOfRegionClosure.html) for details

---
## <a name="nonQ1GYhAL" id="nonQ1GYhAL">ParRebuildRSTask</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される補助クラス (See: [here](no2935ATn.html) for details).

(Major GC の処理後に) 全ての HeapRegion の Remembered Set 情報を作成し直すための AbstractGangTask クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class ParRebuildRSTask: public AbstractGangTask {
```

### 使われ方(Usage)
G1CollectedHeap::do_collection() 内で(のみ)使用されている

### 内部構造(Internal structure)
実際の処理は RebuildRSOutOfRegionClosure に丸投げしている.




### 詳細(Details)
See: [here](../doxygen/classParRebuildRSTask.html) for details

---
## <a name="noIzrF_VeT" id="noIzrF_VeT">SumUsedClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)および Major GC 処理で使用される補助クラス
(See: [here](no2935YzN.html) and [here](no2935ATn.html) for details).

ヒープ中の生きているオブジェクトの合計量 (byte 数) を計算する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class SumUsedClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::recalculate_used() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classSumUsedClosure.html) for details

---
## <a name="noskWJekhr" id="noskWJekhr">SumUsedRegionsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

生きているオブジェクトが存在する HeapRegion の個数を調べる.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    #ifndef PRODUCT
    class SumUsedRegionsClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::recalculate_used_regions() 内で(のみ)使用されている
(なお, この関数自体も #ifndef PRODUCT 時にしか定義されない).




### 詳細(Details)
See: [here](../doxygen/classSumUsedRegionsClosure.html) for details

---
## <a name="no0xevAxFK" id="no0xevAxFK">IterateOopClosureRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

他の OopClosure と組み合わせて使用される Closure クラス.
指定された OopClosure をある HeapRegion 内の全てのポインタフィールドに対して適用する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    // Iterates an OopClosure over all ref-containing fields of objects
    // within a HeapRegion.
    
    class IterateOopClosureRegionClosure: public HeapRegionClosure {
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* G1CollectedHeap::oop_iterate(OopClosure* cl, bool do_perm)
* G1CollectedHeap::oop_iterate(MemRegion mr, OopClosure* cl, bool do_perm)




### 詳細(Details)
See: [here](../doxygen/classIterateOopClosureRegionClosure.html) for details

---
## <a name="noBN-8lXvc" id="noBN-8lXvc">IterateObjectClosureRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

他の ObjectClosure と組み合わせて使用される Closure クラス.
指定された ObjectClosure をある HeapRegion 内の全てのオブジェクトに対して適用する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    // Iterates an ObjectClosure over all objects within a HeapRegion.
    
    class IterateObjectClosureRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::object_iterate() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classIterateObjectClosureRegionClosure.html) for details

---
## <a name="noSk6aOeAU" id="noSk6aOeAU">SpaceClosureRegionClosure</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

他の SpaceClosure と組み合わせて使用される Closure クラス.
指定された SpaceClosure をある HeapRegion に対して適用する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    // Calls a SpaceClosure on a HeapRegion.
    
    class SpaceClosureRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::space_iterate() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classSpaceClosureRegionClosure.html) for details

---
## <a name="noOA3rQgUA" id="noOA3rQgUA">ResetClaimValuesClosure</a>

### 概要(Summary)
G1CollectedHeap の Major GC 処理で使用される補助クラス (See: [here](no2935ATn.html) for details).

(Major GC の処理後に) 全ての HeapRegion の region claim values 情報 (HeapRegion::_claimed フィールド) 
をリセットするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class ResetClaimValuesClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::reset_heap_region_claim_values() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classResetClaimValuesClosure.html) for details

---
## <a name="nokp-dl-4l" id="nokp-dl-4l">CheckClaimValuesClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

全ての HeapRegion の region claim values 情報 (HeapRegion::_claimed フィールド), 
及び Humongous 情報が正しいかどうかのチェックを行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    #ifdef ASSERT
    // This checks whether all regions in the heap have the correct claim
    // value. I also piggy-backed on this a check to ensure that the
    // humongous_start_region() information on "continues humongous"
    // regions is correct.
    
    class CheckClaimValuesClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::check_heap_region_claim_values() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classCheckClaimValuesClosure.html) for details

---
## <a name="nobpMESrkQ" id="nobpMESrkQ">VerifyLivenessOopClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス. 

VerifyObjsInRegionClosure クラス内で使用される補助クラス(Closureクラス).
処理対象のオブジェクトが生きていることをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class VerifyLivenessOopClosure: public OopClosure {
```

### 使われ方(Usage)
VerifyObjsInRegionClosure::do_object() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classVerifyLivenessOopClosure.html) for details

---
## <a name="nor10dvm86" id="nor10dvm86">VerifyObjsInRegionClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

VerifyRegionClosure クラス内で使用される補助クラス(Closureクラス).
「処理対象のオブジェクトが生きていれば, その中のポインタフィールドが指す先も全て生きている」ことをチェックする.
また, Concurrent Mark 開始時以前に確保されたオブジェクトの中で現在も生きているものの量(byte 数)も計算している.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class VerifyObjsInRegionClosure: public ObjectClosure {
```

### 使われ方(Usage)
VerifyRegionClosure::doHeapRegion() 内で(のみ)使用されている




### 詳細(Details)
See: [here](../doxygen/classVerifyObjsInRegionClosure.html) for details

---
## <a name="nohCs-vjJ9" id="nohCs-vjJ9">PrintObjsInRegionClosure</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class PrintObjsInRegionClosure : public ObjectClosure {
```




### 詳細(Details)
See: [here](../doxygen/classPrintObjsInRegionClosure.html) for details

---
## <a name="noVYTRoIgH" id="noVYTRoIgH">VerifyRegionClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

HeapRegion が正しい状態になっていることをチェックするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class VerifyRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::verify() 内で(のみ)使用されている
(より正確には, G1CollectedHeap::verify() 内と,
そこから呼び出される G1ParVerifyTask::work() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classVerifyRegionClosure.html) for details

---
## <a name="nohAXIM_4C" id="nohAXIM_4C">VerifyRootsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

root から指されている先が生きていることをチェックするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class VerifyRootsClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
G1CollectedHeap::verify() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVerifyRootsClosure.html) for details

---
## <a name="noqdtTpUGi" id="noqdtTpUGi">G1ParVerifyTask</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

HeapRegion が正しい状態になっていることをチェックするための AbstractGangTask クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    // This is the task used for parallel heap verification.
    
    class G1ParVerifyTask: public AbstractGangTask {
```

### 使われ方(Usage)
G1CollectedHeap::verify() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1ParVerifyTask.html) for details

---
## <a name="noc75HHpuv" id="noc75HHpuv">PrintRegionClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス(??) (#ifdef ASSERT 時にしか使用されない? #TODO).

処理対象の HeapRegion の情報を出力する Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class PrintRegionClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::print_on_extended() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::print_on()
-&gt; G1CollectedHeap::print_on_extended()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classPrintRegionClosure.html) for details

---
## <a name="noNpgTyG8w" id="noNpgTyG8w">VerifyMarkedObjsClosure</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) (See: VerifyDuringGC).

Concurrent Marking 処理の結果が正しいことをチェックするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class VerifyMarkedObjsClosure: public ObjectClosure {
```

### 使われ方(Usage)
G1CollectedHeap::checkConcurrentMark() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVerifyMarkedObjsClosure.html) for details

---
## <a name="no7djYUaU4" id="no7djYUaU4">FindGCAllocRegion</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef G1_DEBUG 時にしか定義されない).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    #ifdef G1_DEBUG
    class FindGCAllocRegion: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::forget_alloc_region_list() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classFindGCAllocRegion.html) for details

---
## <a name="no0FiI45cN" id="no0FiI45cN">G1IsAliveClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.
G1IsAliveClosure::do_object_b() メソッドが呼ばれると, 処理対象のオブジェクトが生きているかどうかを返す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class G1IsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
G1CollectedHeap::evacuate_collection_set() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1IsAliveClosure.html) for details

---
## <a name="nom5QrhIVU" id="nom5QrhIVU">G1KeepAliveClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class G1KeepAliveClosure: public OopClosure {
```

### 使われ方(Usage)
G1CollectedHeap::evacuate_collection_set() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1KeepAliveClosure.html) for details

---
## <a name="noUOTjl0_A" id="noUOTjl0_A">UpdateRSetDeferred</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC がメモリ不足で失敗した場合に, 
コピーできなかったオブジェクトについて, Remembered Set 情報を作成し直す.

なお, このクラスは develop オプションである G1DeferredRSUpdate が変更されていない場合(= true の場合)に使用される.
G1DeferredRSUpdate オプションが false の場合には, 代わりに UpdateRSetImmediate クラスが使用される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class UpdateRSetDeferred : public OopsInHeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::remove_self_forwarding_pointers() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classUpdateRSetDeferred.html) for details

---
## <a name="nomBxVH4Ra" id="nomBxVH4Ra">RemoveSelfPointerClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

Minor GC がメモリ不足で失敗した場合に, 
コピーできなかったオブジェクトについて, markbit や Remembered Set 情報を作成し直す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RemoveSelfPointerClosure: public ObjectClosure {
```

### 使われ方(Usage)
G1CollectedHeap::remove_self_forwarding_pointers() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classRemoveSelfPointerClosure.html) for details

---
## <a name="noWsif6LhY" id="noWsif6LhY">G1ParEvacuateFollowersClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

処理したオブジェクトから辿れる範囲全てについて再帰的に処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class G1ParEvacuateFollowersClosure : public VoidClosure {
```

### 使われ方(Usage)
G1ParTask::work() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1ParEvacuateFollowersClosure.html) for details

---
## <a name="nopbvNS8Go" id="nopbvNS8Go">G1ParTask</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される補助クラス (See: [here](no2935YzN.html) for details).

実際の Minor GC 処理("Evacuation Pause" 処理) を行う AbstractGangTask クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class G1ParTask : public AbstractGangTask {
```

### 使われ方(Usage)
G1CollectedHeap::evacuate_collection_set() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1ParTask.html) for details

---
## <a name="nodN44Ql2C" id="nodN44Ql2C">SaveMarksClosure</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される Closure クラス (See: [here](no2935YzN.html) for details).

(Minor GC の処理前に) 全ての HeapRegion の top 位置を記録しておく(save_marks)ための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class SaveMarksClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::save_marks() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::do_collection_pause_at_safepoint()
-&gt; G1CollectedHeap::save_marks()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classSaveMarksClosure.html) for details

---
## <a name="no9FY64-0W" id="no9FY64-0W">G1ParCleanupCTTask</a>

### 概要(Summary)
G1CollectedHeap の Minor GC 処理("Evacuation Pause" 処理)で使用される補助クラス (See: [here](no2935YzN.html) for details).

(Minor GC の処理後に) Collection Set 中の HeapRegion の中で Barrier Set をクリアする必要があるもの
(= CollectedHeap::_dirty_cards_region_list 内の HeapRegion) に対し, クリア処理を行う.

(ついでに, Survivor として使われている HeapRegion の Barrier Set を dirty 化する処理も行っている).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class G1ParCleanupCTTask : public AbstractGangTask {
```

### 使われ方(Usage)
G1CollectedHeap::cleanUpCardTable() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
G1CollectedHeap::evacuate_collection_set()
-&gt; G1RemSet::cleanup_after_oops_into_collection_set_do()
   -&gt; G1CollectedHeap::cleanUpCardTable()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classG1ParCleanupCTTask.html) for details

---
## <a name="no3FzkHj0B" id="no3FzkHj0B">G1VerifyCardTableCleanup</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

(Minor GC の処理後に) 以下が成り立っていることをチェックする.

* Survivor として使われている HeapRegion は, 対応する Barrier Set が dirty 化されている.
* その他の HeapRegion は, 対応する Barrier Set がクリアされている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    #ifndef PRODUCT
    class G1VerifyCardTableCleanup: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::cleanUpCardTable() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classG1VerifyCardTableCleanup.html) for details

---
## <a name="noK5HkRWLF" id="noK5HkRWLF">NoYoungRegionsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか使用されない).

処理対象の HeapRegion が New 領域に含まれていない(= HeapRegion::_young_type が NotYoung である)ことをチェックする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class NoYoungRegionsClosure: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::check_young_list_empty() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている
(ただし, どちらも assert() 内でしか使用されていない).

<div class="flow-abst"><pre>
* Minor GC 処理
  G1CollectedHeap::do_collection_pause_at_safepoint()
  -&gt; G1CollectedHeap::check_young_list_empty()

* Major GC 処理
  G1CollectedHeap::do_collection()
  -&gt; G1CollectedHeap::check_young_list_empty()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classNoYoungRegionsClosure.html) for details

---
## <a name="nocLWd0QzX" id="nocLWd0QzX">RegionResetter</a>

### 概要(Summary)
G1CollectedHeap の処理で使用される補助クラス.

(ヒープの縮小時や Major GC の処理後に) 
MasterFreeRegionList や MasterHumongousRegionSet を再構築するための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    class RegionResetter: public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::rebuild_region_lists() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classRegionResetter.html) for details

---
## <a name="nowrBBrH_c" id="nowrBBrH_c">VerifyRegionListsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

MasterFreeRegionList や MasterHumongousRegionSet が正しい状態になっていることをチェックするための Closure クラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
    // Heap region set verification
    
    class VerifyRegionListsClosure : public HeapRegionClosure {
```

### 使われ方(Usage)
G1CollectedHeap::verify_region_sets() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* 
  G1CollectedHeap::verify()
  -&gt; G1CollectedHeap::verify_region_sets()

* 
  G1CollectedHeap::verify_region_sets_optional()
  -&gt; G1CollectedHeap::verify_region_sets()   (&lt;= ただし #ifdef HEAP_REGION_SET_FORCE_VERIFY 時にしか呼び出されない)
</pre></div>

### 備考(Notes)
HEAP_REGION_SET_FORCE_VERIFY は, デフォルトだと #ifdef ASSERT 時にのみ定義される
(このため #ifdef HEAP_REGION_SET_FORCE_VERIFY は #ifdef ASSERT と同義).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/heapRegionSet.hpp))
    // Set verification will be forced either if someone defines
    // HEAP_REGION_SET_FORCE_VERIFY to be 1, or in builds in which
    // asserts are compiled in.
    #ifndef HEAP_REGION_SET_FORCE_VERIFY
    #define HEAP_REGION_SET_FORCE_VERIFY defined(ASSERT)
    #endif // HEAP_REGION_SET_FORCE_VERIFY
```




### 詳細(Details)
See: [here](../doxygen/classVerifyRegionListsClosure.html) for details

---
