---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Major GC の処理 ： UseSerialGC の場合
---
[Up](noVuFF2raP.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Major GC の処理 ： UseSerialGC の場合

--- 
## 概要(Summary)
UseSerialGC の場合, 
GenCollectedHeap の Major GC 処理は, 
TenuredGeneration::collect() を呼び出すことで行われる (See: [here](no28916fAb.html) for details).

実際の Major GC 処理は, 
TenuredGeneration::collect() から呼び出される GenMarkSweep::invoke_at_safepoint() の中に実装されている.
この GenMarkSweep::invoke_at_safepoint() の処理は, 大きく分けると4つのフェーズからなる.

  1. GenMarkSweep::mark_sweep_phase1() で, 全ての生きているオブジェクト(live object)にマークを付ける.
  2. GenMarkSweep::mark_sweep_phase2() で, 各 live object に対して, コンパクション後の新しいアドレスを計算する.
  3. GenMarkSweep::mark_sweep_phase3() で, 各 live object 内のポインタを新しいアドレスに修正する.
  4. GenMarkSweep::mark_sweep_phase4() で, 各 live object を新しいアドレスに移動させる.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
TenuredGeneration::collect()
-&gt; OneContigSpaceCardGeneration::collect()
   -&gt; GenMarkSweep::invoke_at_safepoint()
      -&gt; (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
             -&gt; GenMarkSweep::mark_sweep_phase1()
                (1) strong root から辿り着けるオブジェクト全てに mark を付ける.
                    -&gt; GenCollectedHeap::gen_process_strong_roots()
                       (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::FollowRootClosure)
                       -&gt; (See: <a href="no2114uZg.html">here</a> for details)
                          -&gt; MarkSweep::FollowRootClosure::do_oop()
                             -&gt; MarkSweep::follow_root()
                                まだマークが付いていないオブジェクトであれば, 以下の処理を行う.
                                -&gt; MarkSweep::mark_object() でマークを付ける
                                -&gt; oopDesc::follow_contents()
                                   -&gt; *Klass::oop_follow_contents() で再帰的にポインタを辿ってマークを付けていく.
                                      (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::mark_and_push() で辿るだけ??)
                (1) 以上の処理で見つかった参照オブジェクトを処理する.
                    -&gt; ReferenceProcessor::process_discovered_references()
                       (なお使用するクロージャーは,
                       MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, MarkSweep::FollowStackClosure.
                       AbstractRefProcTaskExecutor は使用しない (NULL を渡す))
                       -&gt; (後述) (See: <a href="no289169tf.html">here</a> for details)
                (1) 最後に, #TODO
                    -&gt; SystemDictionary::do_unloading()
                    -&gt; CodeCache::do_unloading()
                    -&gt;
                    -&gt; MarkSweep::follow_weak_klass_links()
                    -&gt; MarkSweep::follow_mdo_weak_refs()
                    -&gt; StringTable::unlink()
                    -&gt; SymbolTable::unlink()
    
         (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
             -&gt; GenMarkSweep::mark_sweep_phase2()
                -&gt; GenCollectedHeap::prepare_for_compaction()  (New/Old 領域用)
                   -&gt; Generation::prepare_for_compaction()     (&lt;= この関数が New と Old 用に ２回呼ばれる)
                      -&gt; ContiguousSpace::prepare_for_compaction()  (&lt;= この関数は, 各世代で保有している Space の数だけ呼ばれる)
                         -&gt; SCAN_AND_FORWARD
                            -&gt; CompactibleSpace::forward()
                -&gt; Generation::prepare_for_compaction()        (Perm 領域用)
    
         (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.    
             -&gt; GenMarkSweep::mark_sweep_phase3()
                -&gt; CompactingPermGenGen::pre_adjust_pointers()
                -&gt; GenCollectedHeap::gen_process_strong_roots()
                   (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::AdjustPointerClosure)
                   -&gt; (上述)
                      -&gt; MarkSweep::AdjustPointerClosure::do_oop()
                         -&gt; MarkSweep::adjust_pointer()
                -&gt; GenCollectedHeap::gen_process_weak_roots()
                   -&gt;
                -&gt; MarkSweep::adjust_marks()
                -&gt; GenCollectedHeap::generation_iterate()  (New/Old 領域用)
                   (なお使用するクロージャーは GenAdjustPointersClosure)
                   -&gt; GenAdjustPointersClosure::do_generation()   (Old 用)
                      -&gt; Generation::adjust_pointers()
                         -&gt; OneContigSpaceCardGeneration::space_iterate()
                            -&gt; AdjustPointersClosure::do_space()
                               -&gt; CompactibleSpace::adjust_pointers()
                                  -&gt; SCAN_AND_ADJUST_POINTERS
                                     -&gt; oopDesc::adjust_pointers()
                                        -&gt; Klass::oop_adjust_pointers()
                                           (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::adjust_pointer() でポインタを修正するだけ??)
                   -&gt; GenAdjustPointersClosure::do_generation()   (New 用)
                      -&gt; (同上)
                         -&gt; DefNewGeneration::space_iterate()
                            -&gt; AdjustPointersClosure::do_space()  (Eden 用)
                            -&gt; AdjustPointersClosure::do_space()  (From 用)
                            -&gt; AdjustPointersClosure::do_space()  (To 用)
                -&gt; CompactingPermGenGen::adjust_pointers()
                   -&gt; CompactibleSpace::adjust_pointers()
                      -&gt; (同上)
    
         (1) Phase 4: 各 live object を新しいアドレスに移動させる.
             -&gt; GenMarkSweep::mark_sweep_phase4()
                -&gt; Generation::compact()                   (Perm 領域用)
                   -&gt;
                -&gt; GenCollectedHeap::generation_iterate()  (New/Old 領域用)
                   (なお使用するクロージャーは GenCompactClosure)
                   -&gt; (同上)
                      -&gt; GenCompactClosure::do_generation()   (Old 用)
                         -&gt; Generation::compact()
                            -&gt; CompactibleSpace::compact()
                               -&gt; SCAN_AND_COMPACT
                                  -&gt; Copy::aligned_conjoint_words()
                                  -&gt; oopDesc::init_mark()
                      -&gt; GenCompactClosure::do_generation()   (New 用)
                         -&gt; (同上)
                -&gt; Generation::post_compact()               (Perm 領域用)
                   -&gt; #TODO

      -&gt;
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### TenuredGeneration::collect()
See: [here](no28916z7K.html) for details
### OneContigSpaceCardGeneration::collect()
See: [here](no28916AGR.html) for details
### GenMarkSweep::invoke_at_safepoint()
(#Under Construction)
See: [here](no28916nrX.html) for details
### GenMarkSweep::mark_sweep_phase1()
(#Under Construction)
See: [here](no2891601d.html) for details
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
See: [here](no28916nyL.html) for details
### *Klass::oop_follow_contents()
(#Under Construction)

### GenMarkSweep::mark_sweep_phase2()
See: [here](no2891608R.html) for details
### GenCollectedHeap::prepare_for_compaction()
See: [here](no28916bbk.html) for details
### Generation::prepare_for_compaction()
See: [here](no28916olq.html) for details
### CompactibleSpace::prepare_for_compaction()
See: [here](no289161vw.html) for details
### ContiguousSpace::prepare_for_compaction()
See: [here](no28916C62.html) for details
### SCAN_AND_FORWARD マクロ
See: [here](no28916BOM.html) for details
### TenuredSpace::allowed_dead_ratio()
See: [here](no2114ZyF.html) for details
### ContigPermSpace::allowed_dead_ratio()
See: [here](no2114m8L.html) for details
### CompactibleSpace::insert_deadspace()
See: [here](no2114zGS.html) for details

### CompactibleSpace::forward()
(#Under Construction)
See: [here](no28916OYS.html) for details
### GenMarkSweep::mark_sweep_phase3()
See: [here](no28916BHY.html) for details
### MarkSweep::AdjustPointerClosure::do_oop()
See: [here](no28916kHo.html) for details
### MarkSweep::adjust_pointer()
See: [here](no28916xRu.html) for details
### GenCollectedHeap::gen_process_weak_roots()
(#Under Construction)
See: [here](no2114BSf.html) for details
### SharedHeap::process_weak_roots()
(#Under Construction)
See: [here](no2114Ocl.html) for details
### MarkSweep::adjust_marks()
See: [here](no2114Nbe.html) for details
### PreservedMark::adjust_pointer()
See: [here](no2114alk.html) for details

### GenCollectedHeap::generation_iterate()
(#Under Construction)
See: [here](no28916biY.html) for details
### GenAdjustPointersClosure::do_generation()
See: [here](no28916ose.html) for details
### Generation::adjust_pointers()
See: [here](no2891612k.html) for details
### OneContigSpaceCardGeneration::space_iterate()
See: [here](no28916PLx.html) for details
### AdjustPointersClosure::do_space()
See: [here](no28916BVA.html) for details
### CompactibleSpace::adjust_pointers()
See: [here](no28916OfG.html) for details
#### SCAN_AND_ADJUST_POINTERS マクロ
See: [here](no28916bpM.html) for details
### oopDesc::adjust_pointers()
See: [here](no2114mDA.html) for details
### DefNewGeneration::space_iterate()
See: [here](no28916CBr.html) for details
### CompactingPermGenGen::adjust_pointers()
See: [here](no28916ozS.html) for details

### GenMarkSweep::mark_sweep_phase4()
See: [here](no28916ORe.html) for details
### CompactingPermGenGen::post_compact()
See: [here](no211405w.html) for details

### GenCompactClosure::do_generation()
See: [here](no2891619Y.html) for details
### Generation::compact()
See: [here](no28916CIf.html) for details
### CompactibleSpace::compact()
See: [here](no28916PSl.html) for details
#### SCAN_AND_COMPACT マクロ
(#Under Construction)
See: [here](no28916ccr.html) for details

### MarkSweep::restore_marks()
See: [here](no2114ARY.html) for details
### PreservedMark::restore()
See: [here](no2114nvq.html) for details






