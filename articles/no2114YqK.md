---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： Major GC の処理 (UseParallelOldGC 無効時)
---
[Up](no3718vrX.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： ParallelScavengeHeap の場合 ： Major GC の処理 (UseParallelOldGC 無効時)

--- 
## 概要(Summary)
UseParallelOldGC オプションが指定されていない場合, 
ParallelScavengeHeap の Major GC 処理は, 
PSMarkSweep::invoke() を呼び出すことで行われる (See: [here](no3718vrX.html) for details).

実際の Major GC 処理は, 
PSMarkSweep::invoke() から呼び出される PSMarkSweep::invoke_no_policy() の中に実装されている.
この PSMarkSweep::invoke_no_policy() の処理は, 大きく分けると4つのフェーズからなる.

  1. PSMarkSweep::mark_sweep_phase1() で, 全ての生きているオブジェクト(live object)にマークを付ける.
  2. PSMarkSweep::mark_sweep_phase2() で, 各 live object に対して, コンパクション後の新しいアドレスを計算する.
  3. PSMarkSweep::mark_sweep_phase3() で, 各 live object 内のポインタを新しいアドレスに修正する.
  4. PSMarkSweep::mark_sweep_phase4() で, 各 live object を新しいアドレスに移動させる.

なお, ほとんどの処理は PSMarkSweep クラスで実装されている.
ただし, 処理の肝心な部分については PSMarkSweepDecorator クラスで実装されていることが多い.

## 備考(Notes)
(基本的に Serial Old とほぼ同じなので, Serial Old を知ってたら見なくていい気もする...) (See: [here](no2114hPa.html) for details)

## 処理の流れ (概要)(Execution Flows : Summary)
```
-> PSMarkSweep::invoke()
   -> PSScavenge::invoke_no_policy()        (<= ScavengeBeforeFullGC オプションが指定されていれば)
      -> (See: [here](no289165Un.html) for details)
   -> PSMarkSweep::invoke_no_policy()
      -> (1) まず, 前準備をしておく
             ->
             -> ParallelScavengeHeap::accumulate_statistics_all_tlabs()
             -> ParallelScavengeHeap::ensure_parsability()
             ->
             -> BiasedLocking::preserve_marks()
             -> PSMarkSweep::allocate_stacks()
             ->

         (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
             -> PSMarkSweep::mark_sweep_phase1()
                (1) まず, strong root から直接参照されているオブジェクトに mark を付ける.
                    (なお, ここで使用するクロージャーは MarkSweep::MarkAndPushClosure)
                    -> Universe::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> MarkSweep::mark_and_push()
                    -> ReferenceProcessor::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> JNIHandles::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> Threads::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> ObjectSynchronizer::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> FlatProfiler::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> Management::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> JvmtiExport::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                    -> SystemDictionary::always_strong_oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::MarkAndPushClosure::do_oop()
                             -> (同上)
                (1) 次に, mark を付けたオブジェクトから再帰的に辿り付けるオブジェクト全てに mark を付ける.
                    -> MarkSweep::follow_stack()
                (1) 以上の処理で見つかった参照オブジェクト(java.lang.ref オブジェクト)を処理する
                    -> ReferenceProcessor::process_discovered_references()
                (1) 最後に, #TODO
                    -> SystemDictionary::do_unloading()
                    -> CodeCache::do_unloading()
                    ->
                    -> MarkSweep::follow_weak_klass_links()
                    -> MarkSweep::follow_mdo_weak_refs()
                    -> StringTable::unlink()
                    -> SymbolTable::unlink()

         (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
             -> PSMarkSweep::mark_sweep_phase2()
                -> PSOldGen::precompact()          (New/Old 領域用)
                   -> PSMarkSweepDecorator::precompact()  (Old 用)
                   -> PSYoungGen::precompact()            (New 用)
                      -> PSMarkSweepDecorator::precompact()   (Eden 用)
                      -> PSMarkSweepDecorator::precompact()   (From 用)
                      -> PSMarkSweepDecorator::precompact()   (To 用)
                -> PSPermGen::precompact()                (Perm 用)
                   -> PSMarkSweepDecorator::precompact()  (Perm 用)

         (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.
             -> PSMarkSweep::mark_sweep_phase3()
                (1) まず, strong root 内のポインタを修正する
                    (なお, ここで使用するクロージャーは MarkSweep::AdjustPointerClosure)
                    -> Universe::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> MarkSweep::adjust_pointer()
                    -> ReferenceProcessor::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> JNIHandles::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> Threads::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> ObjectSynchronizer::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> FlatProfiler::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> Management::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> JvmtiExport::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> SystemDictionary::always_strong_oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> JNIHandles::weak_oops_do()
                       -> (See: [here](no289169tf.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> CodeCache::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> StringTable::oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                    -> ReferenceProcessor::weak_oops_do()
                       -> (See: [here](no2114GzS.html) for details)
                          -> MarkSweep::AdjustPointerClosure::do_oop()
                             -> (同上)
                (1) 次に, PreservedMark 内のポインタを修正する
                    -> MarkSweep::adjust_marks()
                (1) 最後に, それぞれのヒープ領域について, その領域内の live オブジェクト中のポインタを修正する.
                    -> PSYoungGen::adjust_pointers()               (New 領域用)
                       -> PSMarkSweepDecorator::adjust_pointers()    (Eden 用)
                          -> oopDesc::adjust_pointers()
                             ->
                       -> PSMarkSweepDecorator::adjust_pointers()    (From 用)
                       -> PSMarkSweepDecorator::adjust_pointers()    (To 用)
                    -> PSOldGen::adjust_pointers()                 (Old 領域用)
                       -> PSMarkSweepDecorator::adjust_pointers()
                    -> PSOldGen::adjust_pointers()                 (Perm 領域用)
                       -> (同上)

         (1) Phase 4: 各 live object を新しいアドレスに移動させる.
             -> PSMarkSweep::mark_sweep_phase4()
                -> PSOldGen::compact()                             (Perm 領域用)
                   -> PSMarkSweepDecorator::compact()
                -> PSOldGen::compact()                             (Old 領域用)
                   -> PSMarkSweepDecorator::compact()
                -> PSYoungGen::compact()                           (New 領域用)
                   -> PSMarkSweepDecorator::compact()                (Eden 用)
                   -> PSMarkSweepDecorator::compact()                (From 用)
                   -> PSMarkSweepDecorator::compact()                (To 用)

         (1) GC 処理の後片付けを行う
             -> MarkSweep::restore_marks()
             -> PSMarkSweep::deallocate_stacks()
             ->
             -> PSMarkSweep::absorb_live_data_from_eden()
             ->
             -> BiasedLocking::restore_marks()
             -> ReferenceProcessor::enqueue_discovered_references()
             ->

         (1) GC 結果に基づいて領域長を調整する
             -> (See: [here](novaVS4dhn.html) for details)
             -> ParallelScavengeHeap::resize_all_tlabs()
             -> PSPermGen::compute_new_size()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### PSMarkSweep::invoke()
See: [here](no28916z2A.html) for details
### PSMarkSweep::invoke_no_policy()
(#Under Construction)
See: [here](no28916ABH.html) for details

### BiasedLocking::preserve_marks()
See: [here](no2114l0Q.html) for details
### PSMarkSweep::allocate_stacks()
See: [here](no2114_Id.html) for details

### PSMarkSweep::mark_sweep_phase1()
(#Under Construction)
See: [here](no2114_PR.html) for details
### MarkSweep::MarkAndPushClosure::do_oop()
MarkSweep::mark_and_push() を呼び出すだけ.

```
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    void MarkSweep::MarkAndPushClosure::do_oop(oop* p)       { assert(*p == NULL || (*p)->is_oop(), ""); mark_and_push(p); }
    void MarkSweep::MarkAndPushClosure::do_oop(narrowOop* p) { mark_and_push(p); }
```

### MarkSweep::mark_and_push()
See: [here](no2114Zkd.html) for details
### MarkSweep::follow_stack()
(#Under Construction)
See: [here](no28916WKD.html) for details
### PSMarkSweep::mark_sweep_phase2()
See: [here](no2114Zdp.html) for details
### PSOldGen::precompact()
See: [here](no2114mnv.html) for details
### PSMarkSweepDecorator::precompact()
See: [here](no2114zx1.html) for details
### PSMarkSweepDecorator::advance_destination_decorator()
See: [here](no2114aew.html) for details
### PSMarkSweepDecorator::insert_deadspace()
See: [here](no2114no2.html) for details
### PSYoungGen::precompact()
See: [here](no2114l7E.html) for details
### PSPermGen::precompact()
See: [here](no2114yFL.html) for details

### PSMarkSweep::mark_sweep_phase3()
See: [here](no2114muj.html) for details
### PSYoungGen::adjust_pointers()
See: [here](no2114z4p.html) for details
### PSMarkSweepDecorator::adjust_pointers()
See: [here](no2114NN2.html) for details
### PSOldGen::adjust_pointers()
See: [here](no2114ADw.html) for details

### PSMarkSweep::mark_sweep_phase4()
See: [here](no2114_WF.html) for details
### PSOldGen::compact()
See: [here](no2114MhL.html) for details
### PSMarkSweepDecorator::compact()
See: [here](no2114m1X.html) for details
### PSYoungGen::compact()
See: [here](no2114ZrR.html) for details

### PSMarkSweep::deallocate_stacks()
See: [here](no2114MTj.html) for details
### BiasedLocking::restore_marks()
See: [here](no2114y-W.html) for details
### PSMarkSweep::absorb_live_data_from_eden()
See: [here](no2114NUq.html) for details
### Universe::update_heap_info_at_gc()
See: [here](no2114z_d.html) for details






