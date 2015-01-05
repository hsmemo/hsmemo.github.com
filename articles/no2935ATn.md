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
<div class="flow-abst"><pre>
-&gt; VM_G1CollectForAllocation::doit()
   -&gt; G1CollectedHeap::satisfy_failed_allocation()
      -&gt; G1CollectedHeap::attempt_allocation_at_safepoint()
         -&gt; G1CollectedHeap::humongous_obj_allocate()
            -&gt; G1CollectedHeap::humongous_obj_allocate_find_first()
      -&gt; G1CollectedHeap::expand_and_allocate()
         -&gt; G1CollectedHeap::expand()
            -&gt; VirtualSpace::expand_by()
      -&gt; G1CollectedHeap::do_collection()
         -&gt; G1MarkSweep::invoke_at_safepoint()
            -&gt; (1) まず, 前準備をしておく
                   -&gt;
                   -&gt; ReferenceProcessor::setup_policy()
                   -&gt;
                   -&gt; G1MarkSweep::allocate_stacks()
                   -&gt; BiasedLocking::preserve_marks()

               (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
                   -&gt; G1MarkSweep::mark_sweep_phase1()
                      -&gt; (1) strong root から辿り着けるオブジェクト全てに mark を付ける.
                             -&gt; SharedHeap::process_strong_roots()
                                (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::FollowRootClosure)
                                -&gt; Universe::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; MarkSweep::follow_root()
                                            まだマークが付いていないオブジェクトであれば, 以下の処理を行う.
                                            -&gt; MarkSweep::mark_object() でマークを付ける
                                            -&gt; oopDesc::follow_contents()
                                               -&gt; *Klass::oop_follow_contents() で再帰的にポインタを辿ってマークを付けていく.
                                                  (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::mark_and_push() で辿るだけ??)
                                -&gt; ReferenceProcessor::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; ReferenceProcessor::weak_oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; JNIHandles::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; Threads::possibly_parallel_oops_do() または Threads::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; ObjectSynchronizer::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; FlatProfiler::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; Management::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; JvmtiExport::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; SystemDictionary::oops_do() または SystemDictionary::always_strong_oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; StringTable::oops_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                                -&gt; CodeCache::blobs_do() または CodeCache::scavenge_root_nmethods_do()
                                   -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                                      -&gt; MarkSweep::FollowRootClosure::do_oop()
                                         -&gt; (同上)
                         (1) 以上の処理で見つかった参照オブジェクトを処理する.
                             -&gt; ReferenceProcessor::process_discovered_references()
                                (なお使用するクロージャーは,
                                MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, MarkSweep::FollowStackClosure.
                                AbstractRefProcTaskExecutor は使用しない (NULL を渡す))
                                -&gt; (See: <a href="no289169tf.html">here</a> for details)
                         (1) 最後に, #TODO
                             -&gt;

               (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
                   -&gt; G1MarkSweep::mark_sweep_phase2()
                      -&gt; G1CollectedHeap::heap_region_iterate() (&lt;= 先頭の HeapRegion (= コンパクション先アドレス) を取得)
                         (なお使用するクロージャーは FindFirstRegionClosure)
                         -&gt; HeapRegionSeq::iterate()
                            -&gt; HeapRegionSeq::iterate_from()
                               -&gt; FindFirstRegionClosure::doHeapRegion()
                      -&gt; G1CollectedHeap::heap_region_iterate() (&lt;= HeapRegion 内を処理)
                         (なお使用するクロージャーは G1PrepareCompactClosure)
                         -&gt; (同上)
                            -&gt; G1PrepareCompactClosure::doHeapRegion()
                               -&gt;
                      -&gt; G1PrepareCompactClosure::update_sets()
                         -&gt; G1CollectedHeap::update_sets_after_freeing_regions()
                            -&gt; 
                      -&gt; Generation::prepare_for_compaction()    (&lt;= Perm 領域内を処理)
                         -&gt;

               (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.
                   -&gt; G1MarkSweep::mark_sweep_phase3()
                      -&gt; SharedHeap::process_strong_roots()
                         (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::AdjustPointerClosure)
                         -&gt; (同上)
                            -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                               -&gt; (See: <a href="no2114hPa.html">here</a> for details)
                      -&gt; ReferenceProcessor::weak_oops_do()
                         (なお使用するクロージャーは MarkSweep::AdjustPointerClosure)
                      -&gt; G1CollectedHeap::g1_process_weak_roots()
                         (なお使用するクロージャーは MarkSweep::AdjustPointerClosure)
                         -&gt;
                      -&gt; MarkSweep::adjust_marks()
                      -&gt; G1CollectedHeap::heap_region_iterate()   (&lt;= HeapRegion の処理)
                         (なお使用するクロージャーは G1AdjustPointersClosure)
                         -&gt; (同上)
                            -&gt; G1AdjustPointersClosure::doHeapRegion()
                               -&gt; * Humongous オブジェクトの先頭にあたる場合:
                                    -&gt; oopDesc::adjust_pointers()
                                  * Humongous ではない場合:
                                    -&gt; CompactibleSpace::adjust_pointers()
                                       -&gt; (See: <a href="no2114hPa.html">here</a> for details)

                      -&gt; CompactingPermGenGen::adjust_pointers()  (&lt;= Perm 領域の処理)
                         -&gt; (See: <a href="no2114hPa.html">here</a> for details)

               (1) Phase 4: 各 live object を新しいアドレスに移動させる.
                   -&gt; G1MarkSweep::mark_sweep_phase4()
                      -&gt; Generation::compact()                   (&lt;= Perm 領域の処理)
                      -&gt; G1CollectedHeap::heap_region_iterate()  (&lt;= HeapRegion の処理)
                         (なお使用するクロージャーは G1SpaceCompactClosure)
                         -&gt; (同上)
                            -&gt; G1SpaceCompactClosure::doHeapRegion()
                               -&gt; * Humongous オブジェクトの先頭にあたる場合:
                                    -&gt; HeapRegion::reset_during_compaction()
                                    -&gt; oopDesc::init_mark()
                                  * Humongous ではない場合:
                                    -&gt; CompactibleSpace::compact()
                                       -&gt; (See: <a href="no2114hPa.html">here</a> for details)

               (1) GC 処理の後片付けを行う
                   -&gt; MarkSweep::restore_marks()
                   -&gt; BiasedLocking::restore_marks()
                   -&gt; GenMarkSweep::deallocate_stacks()
                   -&gt; CardTableRS::invalidate()
                   -&gt; Threads::gc_epilogue()
                   -&gt; CodeCache::gc_epilogue()
                   -&gt; JvmtiExport::gc_epilogue()

         -&gt;
         -&gt; G1CollectedHeap::increment_full_collections_completed()
</pre></div>

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







