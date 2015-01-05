---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： G1CollectedHeap の場合 ： Minor GC の処理  
---
[Up](no28916fAb.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： G1CollectedHeap の場合 ： Minor GC の処理  

--- 
## 概要(Summary)
G1CollectedHeap の Minor GC 処理は, 
VM_G1IncCollectionPause::doit() を呼び出すことで行われる (See: [here](no28916fAb.html) for details).

実際の Minor GC 処理は, 
VM_G1IncCollectionPause::doit() から呼び出される G1CollectedHeap::do_collection_pause_at_safepoint() の中に実装されている.
この G1CollectedHeap::do_collection_pause_at_safepoint() の処理は, 大きく分けると5つのフェーズからなる.

  1. G1CollectorPolicy_BestRegionsFirst::choose_collection_set() で, 
     GC 対象の region (collection set) を選択する
  
  2. G1CollectedHeap::g1_process_strong_roots() で, 
     strong root から辿れるオブジェクトを全てコピーする.

  3. G1ParEvacuateFollowersClosure::do_void() で, 
     コピーしたオブジェクトから再帰的に辿れる範囲を全てコピーする.

  4. 参照オブジェクト(java.lang.ref オブジェクト)の処理を行う
  
     (See: [here](no289169tf.html) for details)

  5. 必要があれば concurrent mark 処理を開始させる


## 備考(Notes)
処理の並列化のために GangWorker クラスが使用されている (See: [here](no28916ecK.html) for details).
また, SharedHeap::process_strong_roots() 内では SubTasksDone による並列化も行われている (See: [here](no28916ecK.html) for details).

なお, SharedHeap::process_strong_roots() 内で使用される SubTasksDone は SH_PS_NumElements 個の種別を管理している.
それぞれの root 種別は以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/memory/sharedHeap.cpp))
    // The set of potentially parallel tasks in strong root scanning.
    enum SH_process_strong_roots_tasks {
      SH_PS_Universe_oops_do,
      SH_PS_JNIHandles_oops_do,
      SH_PS_ObjectSynchronizer_oops_do,
      SH_PS_FlatProfiler_oops_do,
      SH_PS_Management_oops_do,
      SH_PS_SystemDictionary_oops_do,
      SH_PS_jvmti_oops_do,
      SH_PS_StringTable_oops_do,
      SH_PS_CodeCache_oops_do,
      // Leave this one last.
      SH_PS_NumElements
    };
```

また, この SubTasksDone のコンストラクタ呼び出しに付いては 
SharedHeap::SharedHeap() 参照 (See: SharedHeap::SharedHeap()).

## 備考(Notes)
strong root から辿る処理において 
(= G1CollectedHeap::g1_process_strong_roots() から呼び出される処理において), 
実際のコピー処理は以下の Closure によって行われる
(concurrent mark を実施する場合は, mark bit への記入が必要になるため, 異なる Closure が使用される).

  * Minor GC 後に concurrent mark を実施する場合:
    * 非 Java Heap 内の root 用 : G1ParScanAndMarkExtRootClosure
    * Perm 内の root 用 : G1ParScanAndMarkPermClosure

  * 〃 を実施しない場合:
    * 非 Java Heap 内の root 用 : G1ParScanExtRootClosure
    * Perm 内の root 用 : G1ParScanPermClosure


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp))
        G1ParScanExtRootClosure         only_scan_root_cl(_g1h, &pss);
        G1ParScanPermClosure            only_scan_perm_cl(_g1h, &pss);
    ...
    
        G1ParScanAndMarkExtRootClosure  scan_mark_root_cl(_g1h, &pss);
        G1ParScanAndMarkPermClosure     scan_mark_perm_cl(_g1h, &pss);
    ...
```


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
VM_G1IncCollectionPause::doit_prologues()
VM_G1IncCollectionPause::doit()
-&gt; G1CollectedHeap::do_collection_pause_at_safepoint()
   -&gt; (1) 前準備を行う
          -&gt; G1CollectedHeap::gc_prologue()
          -&gt; G1CollectedHeap::release_mutator_alloc_region()
          -&gt; G1CollectedHeap::save_marks()
          -&gt; ConcurrentMark::drainAllSATBBuffers()
      
      (1) GC 対象の region (collection set) を選択する
          -&gt; G1CollectorPolicy_BestRegionsFirst::choose_collection_set()

      (1) Copy 先となる region を選択する
          -&gt; G1CollectedHeap::get_gc_alloc_regions()

      (1) GC (evacuation) を実行する
          -&gt; G1CollectedHeap::evacuate_collection_set()
             -&gt; G1ParTask::work()     (&lt;= parallel の場合は WorkGang::run_task() 経由で呼び出される)
                (1) root から直接参照されているオブジェクトを全て処理する.
                    -&gt; G1CollectedHeap::g1_process_strong_roots()
                       -&gt; SharedHeap::process_strong_roots()
                          (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) G1ParCopyClosure. ポインタ配列用は G1ParScanPartialArrayClosure)
                          (&lt;= より正確には, G1ParCopyClosure を BufferingOopClosure や BufferingOopsInGenClosure でラッピングしたものが使われる)
                          -&gt; Universe::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; G1ParCopyClosure::do_oop_nv()
                                      -&gt; G1ParCopyClosure::do_oop_work()
                                         -&gt; G1ParCopyHelper::copy_to_survivor_space()
                                            (1) コピー先の領域にメモリを確保する
                                                -&gt; G1ParScanThreadState::allocate()
                                                   ParGCAllocBuffer::allocate() での確保を試みる.
                                                   失敗したら, G1ParScanThreadState::allocate_slow() での確保を行う
                                            (2) オブジェクトをコピーする
                                            (3) オブジェクト内のポインタを G1ParScanThreadState オブジェクト内に収集する.
                                                * 処理対象がものすごく長いポインタ配列の場合: (全部処理すると遅いのでここでは収集しない)
                                                  -&gt; set_partial_array_mask()
                                                * そうではない場合:
                                                  -&gt; oopDesc::oop_iterate_backwards()
                                                     -&gt; *Klass::oop_oop_iterate_backwards_v() または *Klass::oop_oop_iterate_backwards_nv()
                                                        -&gt; #TODO
                                                           -&gt; G1ParScanClosure::do_oop_nv()
                          -&gt; ReferenceProcessor::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; ReferenceProcessor::weak_oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; JNIHandles::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; Threads::possibly_parallel_oops_do() または Threads::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; ObjectSynchronizer::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; FlatProfiler::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; Management::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; JvmtiExport::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; SystemDictionary::oops_do() または SystemDictionary::always_strong_oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; StringTable::oops_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt; CodeCache::blobs_do() または CodeCache::scavenge_root_nmethods_do()
                             -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                -&gt; G1ParCopyClosure::do_oop()
                                   -&gt; (同上)
                          -&gt;
                       -&gt; ConcurrentMark::oops_do()
                       -&gt; G1RemSet::oops_into_collection_set_do()
                          -&gt; G1RemSet::updateRS()
                             -&gt; G1CollectedHeap::iterate_dirty_card_closure()
                                (なお使用するクロージャーは RefineRecordRefsIntoCSCardTableEntryClosure)
                                -&gt; DirtyCardQueueSet::apply_closure_to_completed_buffer()
                                   -&gt; DirtyCardQueueSet::apply_closure_to_completed_buffer_helper()
                                      -&gt; DirtyCardQueue::apply_closure_to_buffer()
                                         (なお使用するクロージャーは RefineRecordRefsIntoCSCardTableEntryClosure)
                                         -&gt; RefineRecordRefsIntoCSCardTableEntryClosure::do_card_ptr()
                                            -&gt; G1RemSet::concurrentRefineOneCard()
                                               -&gt; ConcurrentG1Refine::cache_insert()
                                               -&gt; G1RemSet::concurrentRefineOneCard_impl()
                                                  -&gt; HeapRegion::oops_on_card_seq_iterate_careful()
                                                     (この場合のクロージャーは, check_for_refs_into_cset 引数が true なので, TriggerClosure をトリガーとして FilterIntoCSClosure を稼働させる InvokeIfNotTriggeredClosure を UpdateRSOrPushRefOopClosure と合成した Mux2Closure (をさらにFilterOutOfRegionClosureでくるんだもの.
                                                      また, UpdateRSOrPushRefOopClosure 内では G1ParPushHeapRSClosure が使用される)
                                                     -&gt; oopDesc::oop_iterate()
                                                        -&gt; FilterOutOfRegionClosure::do_oop()
                                                           -&gt; FilterOutOfRegionClosure::do_oop_nv()
                                                              -&gt; Mux2Closure::do_oop()
                                                                 -&gt; InvokeIfNotTriggeredClosure::do_oop()
                                                                    -&gt; TriggerClosure::do_oop()
                                                                    -&gt; FilterIntoCSClosure::do_oop()
                                                                 -&gt; UpdateRSOrPushRefOopClosure::do_oop()
                                                                    -&gt; UpdateRSOrPushRefOopClosure::do_oop_work()
                                                                       -&gt; * 
                                                                            -&gt; G1ParPushHeapRSClosure::do_oop()
                                                                          * 
                                                                            -&gt; G1RemSet::par_write_ref()
                                                                               -&gt; HeapRegionRemSet::add_reference(OopOrNarrowOopStar from, int tid)
                                                                                  -&gt; OtherRegionsTable::add_reference()
                                                                                     -&gt; OtherRegionsTable::find_region_table()
                                                                                     -&gt; 新しいエントリが必要なら以下の処理を行う.
                                                                                        -&gt; SparsePRT::add_card() (&lt;= これは, 成功すればここでリターン)
                                                                                           -&gt; RSHashTable::add_card()
                                                                                              -&gt; RSHashTable::entry_for_region_ind_create()
                                                                                              -&gt; SparsePRTEntry::add_card()
                                                                                        -&gt; * もういっぱいなので coarse map に追い出して確保する場合
                                                                                             -&gt; OtherRegionsTable::delete_region_table()
                                                                                           * まだ空きがある場合
                                                                                             -&gt; PosParPRT::alloc()
                                                                                     -&gt; PosParPRT::add_reference()
                                                                                        -&gt; PerRegionTable::add_reference()
                                                                                           -&gt; PerRegionTable::add_reference_work()
                                                                                              -&gt; PerRegionTable::add_card_work()
                                                                                                 -&gt; * 
                                                                                                      -&gt; BitMap::par_at_put()
                                                                                                    * 
                                                                                                      -&gt; BitMap::at_put()

                          -&gt; G1RemSet::scanRS()
                             -&gt; G1CollectedHeap::collection_set_iterate_from()
                                (なお使用するクロージャーは ScanRSClosure)
                                -&gt; ScanRSClosure::doHeapRegion()
                                   -&gt; 
                       -&gt; ReferenceProcessor::weak_oops_do()
                       -&gt; ReferenceProcessor::oops_do()

                (2) 処理したオブジェクトから再帰的にたどれる範囲についても全て処理する.
                    -&gt; G1ParEvacuateFollowersClosure::do_void()
                       (なお使用するクロージャーは G1ParCopyClosure (ポインタ配列用は G1ParScanPartialArrayClosure))
                       -&gt; G1ParScanThreadState::deal_with_reference()
                          * 処理対象が, 処理が pending されているポインタ配列の場合:
                            -&gt; G1ParScanPartialArrayClosure::do_oop_nv()
                               -&gt;
                          * それ以外の場合:
                            -&gt; G1ParCopyClosure::do_oop_nv()
                               -&gt; (同上)

                (3)

      (1)

      (1) 必要があれば concurrent mark 処理を開始させる
          -&gt; ConcurrentMark::checkpointRootsInitialPost()
             -&gt; G1CollectedHeap::heap_region_iterate()
                (なお使用するクロージャーは NoteStartOfMarkHRClosure)
                -&gt; HeapRegionSeq::iterate()
                   -&gt; HeapRegionSeq::iterate_from()
                      -&gt; NoteStartOfMarkHRClosure::doHeapRegion()
                         -&gt; HeapRegion::note_start_of_marking()
             -&gt; SATBMarkQueueSet::set_active_all_threads()
          -&gt; G1CollectedHeap::set_marking_started()
          -&gt; G1CollectedHeap::doConcurrentMark()
             -&gt; ConcurrentMarkThread::set_started()
             -&gt; Monitor::notify()                    (&lt;= ここで Concurrent Mark Thread の作業を開始)

VM_G1IncCollectionPause::doit_epilogue()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### VM_G1IncCollectionPause::VM_G1IncCollectionPause()
See: [here](no28916KNC.html) for details
### VM_G1IncCollectionPause::doit_prologues()
(#Under Construction)

### VM_G1IncCollectionPause::doit()
See: [here](no28916XXI.html) for details
### G1CollectedHeap::do_collection_pause_at_safepoint()
(#Under Construction)
See: [here](no28916khO.html) for details
### G1CollectedHeap::gc_prologue()
(#Under Construction)

### G1CollectedHeap::release_mutator_alloc_region()
See: [here](no21493b-a.html) for details
### G1CollectedHeap::save_marks()
(#Under Construction)

### ConcurrentMark::drainAllSATBBuffers()
(#Under Construction)

### ConcurrentMark::newCSet()
(#Under Construction)

### G1CollectorPolicy_BestRegionsFirst::choose_collection_set()
(#Under Construction)
See: [here](no28916-1a.html) for details
### G1CollectorPolicy::predict_base_elapsed_time_ms()
(#Under Construction)

### CollectionSetChooser::getNextMarkedRegion()
(#Under Construction)

### G1CollectorPolicy::predict_region_elapsed_time_ms()
(#Under Construction)

### G1CollectorPolicy::add_to_collection_set()
(#Under Construction)

### G1CollectorPolicy::stop_incremental_cset_building()
(#Under Construction)

### G1CollectorPolicy::count_CS_bytes_used()
(#Under Construction)

### G1CollectedHeap::setup_surviving_young_words()
(#Under Construction)

### G1CollectedHeap::get_gc_alloc_regions()
(#Under Construction)

### G1CollectedHeap::evacuate_collection_set()
(#Under Construction)
See: [here](no28916LAh.html) for details
### G1ParTask::work()
(#Under Construction)
See: [here](no28916YKn.html) for details
### G1CollectedHeap::g1_process_strong_roots()
(#Under Construction)
See: [here](no28916lUt.html) for details

### SharedHeap::process_strong_roots()
(#Under Construction)
See: [here](no28916koC.html) for details
#### 備考(Notes)
処理の概要は以下のコメント参照.
引数で渡された OopClosure (引数名は roots) の do_oop() を全ての root に対して適用する関数.

なお, Perm 内を指しているポインタに付いては, 引数の collecting_perm_gen が false であれば見ない.
この場合, 代わりに Perm 内のオブジェクトは全て live として, 引数で渡された OopsInGenClosure (引数名は perm_blk) を Perm 内の全ての (Perm 外への) 参照に対して適用する.

また, 処理対称にする root 種別は引数で選択することができる (引数名は so).
選択できる root 種別には以下のものがある.

  * SO_None
  * SO_AllClasses
  * SO_SystemClasses
  * SO_Strings
  * SO_CodeCache


```cpp
    ((cite: hotspot/src/share/vm/memory/sharedHeap.hpp))
      // Invoke the "do_oop" method the closure "roots" on all root locations.
      // If "collecting_perm_gen" is false, then roots that may only contain
      // references to permGen objects are not scanned; instead, in that case,
      // the "perm_blk" closure is applied to all outgoing refs in the
      // permanent generation.  The "so" argument determines which of roots
      // the closure is applied to:
      // "SO_None" does none;
      // "SO_AllClasses" applies the closure to all entries in the SystemDictionary;
      // "SO_SystemClasses" to all the "system" classes and loaders;
      // "SO_Strings" applies the closure to all entries in StringTable;
      // "SO_CodeCache" applies the closure to all elements of the CodeCache.
```

### SubTasksDone::clear()
See: [here](no28916-8O.html) for details
### SubTasksDone::is_task_claimed()
See: [here](no28916LHV.html) for details
### SubTasksDone::all_tasks_completed()
See: [here](no28916YRb.html) for details

### BufferingOopClosure::
(#Under Construction)

### BufferingOopsInGenClosure::
(#Under Construction)

### G1ParCopyClosure::do_oop()

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))

(1) G1ParCopyClosure::do_oop_nv() を呼び出すだけ.

      virtual void do_oop(oop* p)       { do_oop_nv(p); }
      virtual void do_oop(narrowOop* p) { do_oop_nv(p); }
```

### G1ParCopyClosure::do_oop_nv()
See: [here](no28916LOJ.html) for details
### G1ParCopyClosure::do_oop_work()
(#Under Construction)
See: [here](no28916YYP.html) for details
### G1ParCopyHelper::copy_to_survivor_space()
(#Under Construction)
See: [here](no28916liV.html) for details
### G1ParPushHeapRSClosure::do_oop()
See: [here](no21493U10.html) for details
### G1ParPushHeapRSClosure::do_oop_nv()
See: [here](no214936nc.html) for details
### G1ParScanThreadState::allocate()
See: [here](no28916ysb.html) for details
### G1ParScanThreadState::allocate_buffer()
(#Under Construction)
See: [here](no28916_2h.html) for details
### ParGCAllocBuffer::allocate()
See: [here](no28916ZLu.html) for details
### G1ParScanThreadState::allocate_slow()
See: [here](no28916MBo.html) for details
### oopDesc::oop_iterate_backwards()
See: [here](no28916mjc.html) for details

### ConcurrentMark::oops_do()
(#Under Construction)

### G1RemSet::oops_into_collection_set_do()
(#Under Construction)
See: [here](no28916mV0.html) for details
### G1RemSet::updateRS()
See: [here](no28916YfD.html) for details
### G1CollectedHeap::iterate_dirty_card_closure()
(#Under Construction)
See: [here](no28916lpJ.html) for details
### DirtyCardQueueSet::apply_closure_to_completed_buffer()
(#Under Construction)

### DirtyCardQueueSet::apply_closure_to_completed_buffer_helper()
See: [here](no2935Ddw.html) for details
### DirtyCardQueue::apply_closure_to_buffer()
See: [here](no2935Qn2.html) for details
### RefineRecordRefsIntoCSCardTableEntryClosure::do_card_ptr()
(#Under Construction)
See: [here](no28916ZSi.html) for details
### G1RemSet::concurrentRefineOneCard()
(#Under Construction)


### Mux2Closure::do_oop()
See: [here](no3420fth.html) for details
### Mux2Closure::do_oop_nv()
See: [here](no3420s3n.html) for details
### InvokeIfNotTriggeredClosure::do_oop()
See: [here](no3420GM0.html) for details
### InvokeIfNotTriggeredClosure::do_oop_nv()
See: [here](no34204VD.html) for details
### TriggerClosure::do_oop()
See: [here](no3420FgJ.html) for details
### TriggerClosure::do_oop_nv()
See: [here](no3420SqP.html) for details
### FilterIntoCSClosure::do_oop()
See: [here](no3420U4b.html) for details
### FilterIntoCSClosure::do_oop_nv()
See: [here](no3420hCi.html) for details
### UpdateRSOrPushRefOopClosure::do_oop()
See: [here](no31276ds.html) for details
### UpdateRSOrPushRefOopClosure::do_oop_work()
See: [here](no3127Hoy.html) for details
### G1RemSet::par_write_ref()
See: [here](no31275xB.html) for details
### HeapRegionRemSet::add_reference(OopOrNarrowOopStar from, int tid)
See: [here](no3127G8H.html) for details
### OtherRegionsTable::add_reference()
See: [here](no3127TGO.html) for details
### SparsePRT::add_card()
See: [here](no3127gQU.html) for details
### RSHashTable::add_card()
See: [here](no3127taa.html) for details
### RSHashTable::entry_for_region_ind_create()
(#Under Construction)

### SparsePRTEntry::add_card()
See: [here](no31276kg.html) for details
### OtherRegionsTable::delete_region_table()
See: [here](no3127Hvm.html) for details
### PosParPRT::par_expand()
See: [here](no3127U5s.html) for details
### PosParPRT::add_reference()
See: [here](no3127hDz.html) for details
### PerRegionTable::add_reference()
See: [here](no3127TNC.html) for details
### PerRegionTable::add_reference_work()
See: [here](no3127gXI.html) for details
### PerRegionTable::add_card_work()
See: [here](no3127thO.html) for details
### BitMap::par_at_put()
See: [here](no31276rU.html) for details
### BitMap::at_put()
See: [here](no3127H2a.html) for details
### BitMap::set_bit()


### BitMap::clear_bit()


### G1RemSet::scanRS()
(#Under Construction)
See: [here](no28916zmu.html) for details
### G1CollectedHeap::collection_set_iterate_from()
(#Under Construction)
See: [here](no28916Ax0.html) for details
### ScanRSClosure::doHeapRegion()
(#Under Construction)
See: [here](no28916y6D.html) for details

### G1ParEvacuateFollowersClosure::do_void()
(#Under Construction)
See: [here](no28916_EK.html) for details
### G1ParScanThreadState::deal_with_reference()
See: [here](no28916ZZW.html) for details
### G1ParScanPartialArrayClosure::do_oop()
See: [here](no21493XNm.html) for details
### G1ParScanPartialArrayClosure::do_oop_nv()
See: [here](no21493XUa.html) for details

### G1CollectedHeap::in_cset_fast_test()
See: [here](no21493-AV.html) for details
### G1CollectedHeap::free_collection_set()
(#Under Construction)

### G1CollectorPolicy::clear_collection_set()
(#Under Construction)

### G1CollectedHeap::cleanup_surviving_young_words()
(#Under Construction)

### G1CollectorPolicy::start_incremental_cset_building()
(#Under Construction)

### YoungList::reset_auxilary_lists()
(#Under Construction)
See: [here](no28916xrU.html) for details
### ConcurrentMark::checkpointRootsInitialPost()
(#Under Construction)
See: [here](no2935bvu.html) for details
### NoteStartOfMarkHRClosure::doHeapRegion()
See: [here](no2935o50.html) for details
### HeapRegion::note_start_of_marking()
(#Under Construction)
See: [here](no2935aDE.html) for details
### G1CollectedHeap::set_marking_started()
See: [here](no2935nNK.html) for details
### G1CollectedHeap::doConcurrentMark()
See: [here](no2935BiW.html) for details
### ConcurrentMarkThread::set_started()
See: [here](no29350XQ.html) for details
### G1CollectedHeap::allocate_dummy_regions()
(#Under Construction)

### G1CollectedHeap::init_mutator_alloc_region()
(#Under Construction)

### G1CollectedHeap::gc_epilogue()
(#Under Construction)

### VM_G1IncCollectionPause::doit_epilogue()
(#Under Construction)








