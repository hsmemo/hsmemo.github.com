---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： G1CollectedHeap の場合 ： Major GC の処理  
---
[Up](no28916fAb.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： G1CollectedHeap の場合 ： Major GC の処理  

--- 
## 概要(Summary)
G1CollectedHeap の Major GC 処理は, 
VM_G1CollectForAllocation::doit() を呼び出すことで行われる (See: [here](no28916fAb.html) for details).

実際の Major GC 処理は, 
VM_G1CollectForAllocation::doit() から呼び出される G1MarkSweep::invoke_at_safepoint() の中に実装されている.
この G1MarkSweep::invoke_at_safepoint() の処理は, 大きく分けると4つのフェーズからなる.

  1. G1MarkSweep::mark_sweep_phase1() で, 全ての生きているオブジェクト(live object)にマークを付ける.
  2. G1MarkSweep::mark_sweep_phase2() で, 各 live object に対して, コンパクション後の新しいアドレスを計算する.
  3. G1MarkSweep::mark_sweep_phase3() で, 各 live object 内のポインタを新しいアドレスに修正する.
  4. G1MarkSweep::mark_sweep_phase4() で, 各 live object を新しいアドレスに移動させる.

なお, ほとんどの処理は G1MarkSweep クラスで実装されている.

## 備考(Notes)
(基本的に Serial Old とほぼ同じなので, Serial Old を知ってたら見なくていい気もする...) (See: [here](no2114hPa.html) for details)

## 処理の流れ (概要)(Execution Flows : Summary)
```
-> VM_G1CollectForAllocation::doit()
   -> G1CollectedHeap::satisfy_failed_allocation()
      -> G1CollectedHeap::attempt_allocation_at_safepoint()
         -> G1CollectedHeap::humongous_obj_allocate()
            -> G1CollectedHeap::humongous_obj_allocate_find_first()
      -> G1CollectedHeap::expand_and_allocate()
         -> G1CollectedHeap::expand()
            -> VirtualSpace::expand_by()
      -> G1CollectedHeap::do_collection()
         -> G1MarkSweep::invoke_at_safepoint()
            -> (1) まず, 前準備をしておく
                   ->
                   -> ReferenceProcessor::setup_policy()
                   ->
                   -> G1MarkSweep::allocate_stacks()
                   -> BiasedLocking::preserve_marks()

               (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
                   -> G1MarkSweep::mark_sweep_phase1()
                      -> (1) strong root から辿り着けるオブジェクト全てに mark を付ける.
                             -> SharedHeap::process_strong_roots()
                                (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::FollowRootClosure)
                                -> Universe::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> MarkSweep::follow_root()
                                            まだマークが付いていないオブジェクトであれば, 以下の処理を行う.
                                            -> MarkSweep::mark_object() でマークを付ける
                                            -> oopDesc::follow_contents()
                                               -> *Klass::oop_follow_contents() で再帰的にポインタを辿ってマークを付けていく.
                                                  (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::mark_and_push() で辿るだけ??)
                                -> ReferenceProcessor::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> ReferenceProcessor::weak_oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> JNIHandles::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> Threads::possibly_parallel_oops_do() または Threads::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> ObjectSynchronizer::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> FlatProfiler::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> Management::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> JvmtiExport::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> SystemDictionary::oops_do() または SystemDictionary::always_strong_oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> StringTable::oops_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                                -> CodeCache::blobs_do() または CodeCache::scavenge_root_nmethods_do()
                                   -> (See: [here](no2114GzS.html) for details)
                                      -> MarkSweep::FollowRootClosure::do_oop()
                                         -> (同上)
                         (1) 以上の処理で見つかった参照オブジェクトを処理する.
                             -> ReferenceProcessor::process_discovered_references()
                                (なお使用するクロージャーは,
                                MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, MarkSweep::FollowStackClosure.
                                AbstractRefProcTaskExecutor は使用しない (NULL を渡す))
                                -> (See: [here](no289169tf.html) for details)
                         (1) 最後に, #TODO
                             ->

               (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
                   -> G1MarkSweep::mark_sweep_phase2()
                      -> G1CollectedHeap::heap_region_iterate() (<= 先頭の HeapRegion (= コンパクション先アドレス) を取得)
                         (なお使用するクロージャーは FindFirstRegionClosure)
                         -> HeapRegionSeq::iterate()
                            -> HeapRegionSeq::iterate_from()
                               -> FindFirstRegionClosure::doHeapRegion()
                      -> G1CollectedHeap::heap_region_iterate() (<= HeapRegion 内を処理)
                         (なお使用するクロージャーは G1PrepareCompactClosure)
                         -> (同上)
                            -> G1PrepareCompactClosure::doHeapRegion()
                               ->
                      -> G1PrepareCompactClosure::update_sets()
                         -> G1CollectedHeap::update_sets_after_freeing_regions()
                            -> 
                      -> Generation::prepare_for_compaction()    (<= Perm 領域内を処理)
                         ->

               (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.
                   -> G1MarkSweep::mark_sweep_phase3()
                      -> SharedHeap::process_strong_roots()
                         (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::AdjustPointerClosure)
                         -> (同上)
                            -> MarkSweep::AdjustPointerClosure::do_oop()
                               -> (See: [here](no2114hPa.html) for details)
                      -> ReferenceProcessor::weak_oops_do()
                         (なお使用するクロージャーは MarkSweep::AdjustPointerClosure)
                      -> G1CollectedHeap::g1_process_weak_roots()
                         (なお使用するクロージャーは MarkSweep::AdjustPointerClosure)
                         ->
                      -> MarkSweep::adjust_marks()
                      -> G1CollectedHeap::heap_region_iterate()   (<= HeapRegion の処理)
                         (なお使用するクロージャーは G1AdjustPointersClosure)
                         -> (同上)
                            -> G1AdjustPointersClosure::doHeapRegion()
                               -> * Humongous オブジェクトの先頭にあたる場合:
                                    -> oopDesc::adjust_pointers()
                                  * Humongous ではない場合:
                                    -> CompactibleSpace::adjust_pointers()
                                       -> (See: [here](no2114hPa.html) for details)

                      -> CompactingPermGenGen::adjust_pointers()  (<= Perm 領域の処理)
                         -> (See: [here](no2114hPa.html) for details)

               (1) Phase 4: 各 live object を新しいアドレスに移動させる.
                   -> G1MarkSweep::mark_sweep_phase4()
                      -> Generation::compact()                   (<= Perm 領域の処理)
                      -> G1CollectedHeap::heap_region_iterate()  (<= HeapRegion の処理)
                         (なお使用するクロージャーは G1SpaceCompactClosure)
                         -> (同上)
                            -> G1SpaceCompactClosure::doHeapRegion()
                               -> * Humongous オブジェクトの先頭にあたる場合:
                                    -> HeapRegion::reset_during_compaction()
                                    -> oopDesc::init_mark()
                                  * Humongous ではない場合:
                                    -> CompactibleSpace::compact()
                                       -> (See: [here](no2114hPa.html) for details)

               (1) GC 処理の後片付けを行う
                   -> MarkSweep::restore_marks()
                   -> BiasedLocking::restore_marks()
                   -> GenMarkSweep::deallocate_stacks()
                   -> CardTableRS::invalidate()
                   -> Threads::gc_epilogue()
                   -> CodeCache::gc_epilogue()
                   -> JvmtiExport::gc_epilogue()

         ->
         -> G1CollectedHeap::increment_full_collections_completed()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### VM_G1CollectForAllocation::doit()
See: [here](no28916mXe.html) for details
### G1CollectedHeap::satisfy_failed_allocation()
See: [here](no2935zIh.html) for details
### G1CollectedHeap::attempt_allocation_at_safepoint()
See: [here](no2935AhP.html) for details
### G1CollectedHeap::humongous_obj_allocate()
(#Under Construction)
See: [here](no21493Pqe.html) for details
### G1CollectedHeap::humongous_obj_allocate_find_first()
(#Under Construction)


### G1CollectedHeap::expand_and_allocate()
See: [here](no21493BfR.html) for details
### G1CollectedHeap::expand()
(#Under Construction)
See: [here](no21493CSw.html) for details
### VirtualSpace::expand_by()
See: [here](no31977tgs.html) for details
### G1CollectedHeap::do_collection()
(#Under Construction)
See: [here](no2935MqO.html) for details
### G1CollectedHeap::wait_while_free_regions_coming()
See: [here](no2935oHd.html) for details
### G1MarkSweep::invoke_at_safepoint()
(#Under Construction)
See: [here](no2935m-a.html) for details
### G1MarkSweep::mark_sweep_phase1()
(#Under Construction)
See: [here](no2935Ndt.html) for details
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

### MarkSweep::FollowRootClosure::do_oop()
See: [here](no28916BAk.html) for details
### MarkSweep::follow_root()
See: [here](no28916OKq.html) for details
### MarkSweep::mark_object()
See: [here](no28916oe2.html) for details
### markOopDesc::must_be_preserved()
See: [here](no28916aoF.html) for details
### MarkSweep::preserve_mark()
See: [here](no28916KBE.html) for details

### oopDesc::follow_contents()
(#Under Construction)
See: [here](no28916nyL.html) for details
### *Klass::oop_follow_contents()
(#Under Construction)


### G1MarkSweep::mark_sweep_phase2()
See: [here](no2935anz.html) for details
### G1CollectedHeap::heap_region_iterate()
See: [here](no2935Z7I.html) for details
### HeapRegionSeq::iterate()
See: [here](no2935mFP.html) for details
### HeapRegionSeq::iterate_from(HeapRegion* r, HeapRegionClosure* blk)
See: [here](no2935zPV.html) for details
### FindFirstRegionClosure::doHeapRegion()
See: [here](no2935Nkh.html) for details
### G1PrepareCompactClosure::doHeapRegion()
(#Under Construction)
See: [here](no2935aun.html) for details
### G1PrepareCompactClosure::update_sets()
See: [here](no2935n4t.html) for details
### G1CollectedHeap::update_sets_after_freeing_regions()
(#Under Construction)
See: [here](no29350C0.html) for details
### G1MarkSweep::mark_sweep_phase3()
(#Under Construction)
See: [here](no2935MxC.html) for details
### G1CollectedHeap::g1_process_weak_roots()
(#Under Construction)

### G1AdjustPointersClosure::doHeapRegion()
See: [here](no2935Aab.html) for details
### G1MarkSweep::mark_sweep_phase4()
See: [here](no2935mMD.html) for details
### G1SpaceCompactClosure::doHeapRegion()
See: [here](no2935zWJ.html) for details
### G1CollectedHeap::increment_full_collections_completed()
See: [here](no2935Z0U.html) for details







