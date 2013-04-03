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
```
TenuredGeneration::collect()
-> OneContigSpaceCardGeneration::collect()
   -> GenMarkSweep::invoke_at_safepoint()
      -> (1) Phase 1: 全ての生きているオブジェクト(live object)にマークを付ける.
             -> GenMarkSweep::mark_sweep_phase1()
                (1) strong root から辿り着けるオブジェクト全てに mark を付ける.
                    -> GenCollectedHeap::gen_process_strong_roots()
                       (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::FollowRootClosure)
                       -> (See: [here](no2114uZg.html) for details)
                          -> MarkSweep::FollowRootClosure::do_oop()
                             -> MarkSweep::follow_root()
                                まだマークが付いていないオブジェクトであれば, 以下の処理を行う.
                                -> MarkSweep::mark_object() でマークを付ける
                                -> oopDesc::follow_contents()
                                   -> *Klass::oop_follow_contents() で再帰的にポインタを辿ってマークを付けていく.
                                      (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::mark_and_push() で辿るだけ??)
                (1) 以上の処理で見つかった参照オブジェクトを処理する.
                    -> ReferenceProcessor::process_discovered_references()
                       (なお使用するクロージャーは,
                       MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, MarkSweep::FollowStackClosure.
                       AbstractRefProcTaskExecutor は使用しない (NULL を渡す))
                       -> (後述) (See: [here](no289169tf.html) for details)
                (1) 最後に, #TODO
                    -> SystemDictionary::do_unloading()
                    -> CodeCache::do_unloading()
                    ->
                    -> MarkSweep::follow_weak_klass_links()
                    -> MarkSweep::follow_mdo_weak_refs()
                    -> StringTable::unlink()
                    -> SymbolTable::unlink()
    
         (1) Phase 2: 各 live object に対して, コンパクション後の新しいアドレスを計算する.
             -> GenMarkSweep::mark_sweep_phase2()
                -> GenCollectedHeap::prepare_for_compaction()  (New/Old 領域用)
                   -> Generation::prepare_for_compaction()     (<= この関数が New と Old 用に ２回呼ばれる)
                      -> ContiguousSpace::prepare_for_compaction()  (<= この関数は, 各世代で保有している Space の数だけ呼ばれる)
                         -> SCAN_AND_FORWARD
                            -> CompactibleSpace::forward()
                -> Generation::prepare_for_compaction()        (Perm 領域用)
    
         (1) Phase 3: 各 live object 内のポインタを新しいアドレスに修正する.    
             -> GenMarkSweep::mark_sweep_phase3()
                -> CompactingPermGenGen::pre_adjust_pointers()
                -> GenCollectedHeap::gen_process_strong_roots()
                   (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) MarkSweep::AdjustPointerClosure)
                   -> (上述)
                      -> MarkSweep::AdjustPointerClosure::do_oop()
                         -> MarkSweep::adjust_pointer()
                -> GenCollectedHeap::gen_process_weak_roots()
                   ->
                -> MarkSweep::adjust_marks()
                -> GenCollectedHeap::generation_iterate()  (New/Old 領域用)
                   (なお使用するクロージャーは GenAdjustPointersClosure)
                   -> GenAdjustPointersClosure::do_generation()   (Old 用)
                      -> Generation::adjust_pointers()
                         -> OneContigSpaceCardGeneration::space_iterate()
                            -> AdjustPointersClosure::do_space()
                               -> CompactibleSpace::adjust_pointers()
                                  -> SCAN_AND_ADJUST_POINTERS
                                     -> oopDesc::adjust_pointers()
                                        -> Klass::oop_adjust_pointers()
                                           (#TODO  クラス毎に少しずつ違うけど, 基本的には MarkSweep::adjust_pointer() でポインタを修正するだけ??)
                   -> GenAdjustPointersClosure::do_generation()   (New 用)
                      -> (同上)
                         -> DefNewGeneration::space_iterate()
                            -> AdjustPointersClosure::do_space()  (Eden 用)
                            -> AdjustPointersClosure::do_space()  (From 用)
                            -> AdjustPointersClosure::do_space()  (To 用)
                -> CompactingPermGenGen::adjust_pointers()
                   -> CompactibleSpace::adjust_pointers()
                      -> (同上)
    
         (1) Phase 4: 各 live object を新しいアドレスに移動させる.
             -> GenMarkSweep::mark_sweep_phase4()
                -> Generation::compact()                   (Perm 領域用)
                   ->
                -> GenCollectedHeap::generation_iterate()  (New/Old 領域用)
                   (なお使用するクロージャーは GenCompactClosure)
                   -> (同上)
                      -> GenCompactClosure::do_generation()   (Old 用)
                         -> Generation::compact()
                            -> CompactibleSpace::compact()
                               -> SCAN_AND_COMPACT
                                  -> Copy::aligned_conjoint_words()
                                  -> oopDesc::init_mark()
                      -> GenCompactClosure::do_generation()   (New 用)
                         -> (同上)
                -> Generation::post_compact()               (Perm 領域用)
                   -> #TODO

      ->
```

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






