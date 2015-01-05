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
<div class="flow-abst"><pre>
-&gt; PSMarkSweep::invoke()
   -&gt; PSScavenge::invoke_no_policy()        (&lt;= ScavengeBeforeFullGC オプションが指定されていれば)
      -&gt; (See: <a href="no289165Un.html">here</a> for details)
   -&gt; PSMarkSweep::invoke_no_policy()
      -&gt; (1) まず, 前準備をしておく
             -&gt;
             -&gt; ParallelScavengeHeap::accumulate_statistics_all_tlabs()
             -&gt; ParallelScavengeHeap::ensure_parsability()
             -&gt;
             -&gt; BiasedLocking::preserve_marks()
             -&gt; PSMarkSweep::allocate_stacks()
             -&gt;

         (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
             -&gt; PSMarkSweep::mark_sweep_phase1()
                (1) まず, strong root から直接参照されているオブジェクトに mark を付ける.
                    (なお, ここで使用するクロージャーは MarkSweep::MarkAndPushClosure)
                    -&gt; Universe::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; MarkSweep::mark_and_push()
                    -&gt; ReferenceProcessor::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; JNIHandles::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; Threads::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; ObjectSynchronizer::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; FlatProfiler::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; Management::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; JvmtiExport::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                    -&gt; SystemDictionary::always_strong_oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::MarkAndPushClosure::do_oop()
                             -&gt; (同上)
                (1) 次に, mark を付けたオブジェクトから再帰的に辿り付けるオブジェクト全てに mark を付ける.
                    -&gt; MarkSweep::follow_stack()
                (1) 以上の処理で見つかった参照オブジェクト(java.lang.ref オブジェクト)を処理する
                    -&gt; ReferenceProcessor::process_discovered_references()
                (1) 最後に, #TODO
                    -&gt; SystemDictionary::do_unloading()
                    -&gt; CodeCache::do_unloading()
                    -&gt;
                    -&gt; MarkSweep::follow_weak_klass_links()
                    -&gt; MarkSweep::follow_mdo_weak_refs()
                    -&gt; StringTable::unlink()
                    -&gt; SymbolTable::unlink()

         (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
             -&gt; PSMarkSweep::mark_sweep_phase2()
                -&gt; PSOldGen::precompact()          (New/Old 領域用)
                   -&gt; PSMarkSweepDecorator::precompact()  (Old 用)
                   -&gt; PSYoungGen::precompact()            (New 用)
                      -&gt; PSMarkSweepDecorator::precompact()   (Eden 用)
                      -&gt; PSMarkSweepDecorator::precompact()   (From 用)
                      -&gt; PSMarkSweepDecorator::precompact()   (To 用)
                -&gt; PSPermGen::precompact()                (Perm 用)
                   -&gt; PSMarkSweepDecorator::precompact()  (Perm 用)

         (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.
             -&gt; PSMarkSweep::mark_sweep_phase3()
                (1) まず, strong root 内のポインタを修正する
                    (なお, ここで使用するクロージャーは MarkSweep::AdjustPointerClosure)
                    -&gt; Universe::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; MarkSweep::adjust_pointer()
                    -&gt; ReferenceProcessor::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; JNIHandles::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; Threads::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; ObjectSynchronizer::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; FlatProfiler::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; Management::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; JvmtiExport::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; SystemDictionary::always_strong_oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; JNIHandles::weak_oops_do()
                       -&gt; (See: <a href="no289169tf.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; CodeCache::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; StringTable::oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                    -&gt; ReferenceProcessor::weak_oops_do()
                       -&gt; (See: <a href="no2114GzS.html">here</a> for details)
                          -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                             -&gt; (同上)
                (1) 次に, PreservedMark 内のポインタを修正する
                    -&gt; MarkSweep::adjust_marks()
                (1) 最後に, それぞれのヒープ領域について, その領域内の live オブジェクト中のポインタを修正する.
                    -&gt; PSYoungGen::adjust_pointers()               (New 領域用)
                       -&gt; PSMarkSweepDecorator::adjust_pointers()    (Eden 用)
                          -&gt; oopDesc::adjust_pointers()
                             -&gt;
                       -&gt; PSMarkSweepDecorator::adjust_pointers()    (From 用)
                       -&gt; PSMarkSweepDecorator::adjust_pointers()    (To 用)
                    -&gt; PSOldGen::adjust_pointers()                 (Old 領域用)
                       -&gt; PSMarkSweepDecorator::adjust_pointers()
                    -&gt; PSOldGen::adjust_pointers()                 (Perm 領域用)
                       -&gt; (同上)

         (1) Phase 4: 各 live object を新しいアドレスに移動させる.
             -&gt; PSMarkSweep::mark_sweep_phase4()
                -&gt; PSOldGen::compact()                             (Perm 領域用)
                   -&gt; PSMarkSweepDecorator::compact()
                -&gt; PSOldGen::compact()                             (Old 領域用)
                   -&gt; PSMarkSweepDecorator::compact()
                -&gt; PSYoungGen::compact()                           (New 領域用)
                   -&gt; PSMarkSweepDecorator::compact()                (Eden 用)
                   -&gt; PSMarkSweepDecorator::compact()                (From 用)
                   -&gt; PSMarkSweepDecorator::compact()                (To 用)

         (1) GC 処理の後片付けを行う
             -&gt; MarkSweep::restore_marks()
             -&gt; PSMarkSweep::deallocate_stacks()
             -&gt;
             -&gt; PSMarkSweep::absorb_live_data_from_eden()
             -&gt;
             -&gt; BiasedLocking::restore_marks()
             -&gt; ReferenceProcessor::enqueue_discovered_references()
             -&gt;

         (1) GC 結果に基づいて領域長を調整する
             -&gt; (See: <a href="novaVS4dhn.html">here</a> for details)
             -&gt; ParallelScavengeHeap::resize_all_tlabs()
             -&gt; PSPermGen::compute_new_size()
</pre></div>


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

```cpp
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






