---
layout: default
title: Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Minor GC の処理 ： UseSerialGC の場合
---
[Up](nowVKc9k-r.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリの確保処理 (GC 処理) ： slow-path の処理 (4) GC 処理 ： GenCollectedHeap の場合 ： Minor GC の処理 ： UseSerialGC の場合

--- 
## 概要(Summary)
UseSerialGC の場合, 
GenCollectedHeap の Minor GC 処理は, 
DefNewGeneration::collect() を呼び出すことで行われる (See: [here](no28916sKh.html) for details).

この DefNewGeneration::collect() の処理は, 大きく分けると3つのフェーズからなる.

  1. strong root から辿れるオブジェクトを全てコピーする.
  
     この処理は GenCollectedHeap::gen_process_strong_roots() で行われる.

  2. コピーしたオブジェクトから再帰的に辿れる範囲を全てコピーする.
  
     この処理は DefNewGeneration::FastEvacuateFollowersClosure::do_void() で行われる.
     
  3. 参照オブジェクト(java.lang.ref オブジェクト)の処理を行う
  
     (See: [here](no289169tf.html) for details)

## 備考(Notes)
なお, 以下の関数群は使用する Closure の型が(マクロを用いて)テンプレート化されており,
それぞれの Closure 用の関数が存在する.

  * GenCollectedHeap::oop_since_save_marks_iterate()
  * DefNewGeneration::oop_since_save_marks_iterate_*()
  * OneContigSpaceCardGeneration::oop_since_save_marks_iterate_*()
  * ContiguousSpace::oop_since_save_marks_iterate_*()
  * oopDesc::oop_iterate()
  * *Klass::oop_oop_iterate(),  *Klass::oop_oop_iterate_*()

DefNewGeneration と OneContigSpaceCardGeneration, 及び
ContiguousSpace と *Klass のメソッドについては,
メソッド名の最後が _nv であれば, 特定の Closure 向けにインスタンス化されたものとなっている
(逆にメソッド名の最後が _v であれば, 一般の Closure 向けとなっている).

(詳しくは ALL_SINCE_SAVE_MARKS_CLOSURES マクロを参照)

## 処理の流れ (概要)(Execution Flows : Summary)
```
DefNewGeneration::collect()
-> (1) root から直接参照されているオブジェクトを全て処理する.
       -> GenCollectedHeap::gen_process_strong_roots()
          -> SharedHeap::process_strong_roots()
             (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) FastScanClosure)
             -> Universe::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> FastScanClosure::do_oop_work()
                         -> DefNewGeneration::copy_to_survivor_space()
                            * オブジェクトの年齢が昇格年齢未満の場合:
                              -> ContiguousSpace::allocate()
                            * オブジェクトの年齢が昇格年齢以上, もしくは To 領域に確保できなかった場合:
                              -> Generation::promote()
                                 -> Generation::allocate()  (<= サブクラスでオーバーライドされている. UseSerialGC の場合は OneContigSpaceCardGeneration::allocate() が呼ばれる)
                                     -> OneContigSpaceCardGeneration::allocate()   (Old 領域内に確保)
                                 -> GenCollectedHeap::handle_failed_promotion() (<= 確保に失敗した場合に呼び出される)
                              -> DefNewGeneration::handle_promotion_failure()   (<= 昇格に失敗した場合に呼び出される)
                                 -> DefNewGeneration::drain_promo_failure_scan_stack()  (中途半端な状態で残されているポインタを修正する処理)
             -> ReferenceProcessor::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> ReferenceProcessor::weak_oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> JNIHandles::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> Threads::possibly_parallel_oops_do() または Threads::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> ObjectSynchronizer::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> FlatProfiler::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> Management::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> JvmtiExport::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> SystemDictionary::oops_do() または SystemDictionary::always_strong_oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> StringTable::oops_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)
             -> CodeCache::blobs_do() または CodeCache::scavenge_root_nmethods_do()
                -> (See: [here](no2114GzS.html) for details)
                   -> FastScanClosure::do_oop()
                      -> (同上)

          ->
    
   (2) 処理したオブジェクトから再帰的にたどれる範囲についても全て処理する.
       -> DefNewGeneration::FastEvacuateFollowersClosure::do_void()
          -> GenCollectedHeap::oop_since_save_marks_iterate()
             (未処理のオブジェクトは To 領域や Old 領域の末尾に溜まっているので, 処理済みの位置を表すポインタが全ての領域の末尾と一致するまでループ)
             (なお使用するクロージャーは (Perm領域用/非Perm領域用のどちらも) FastScanClosure)
             -> DefNewGeneration::oop_since_save_marks_iterate_v()  または  DefNewGeneration::oop_since_save_marks_iterate_nv()                         (New 領域を処理)
                -> ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()                          (Eden 領域を処理)
                   -> oopDesc::oop_iterate()
                      -> *Klass::oop_oop_iterate() または *Klass::oop_oop_iterate_v() または *Klass::oop_oop_iterate_nv()
                         -> #TODO
                            -> FastScanClosure::do_oop()
                               -> (同上)
                -> ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()                          (To 領域を処理)
                -> ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()                          (From 領域を処理)
             -> OneContigSpaceCardGeneration::oop_since_save_marks_iterate_v()  または  OneContigSpaceCardGeneration::oop_since_save_marks_iterate_nv() (Old 領域を処理)
                -> ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()
             -> OneContigSpaceCardGeneration::oop_since_save_marks_iterate_v()  または  OneContigSpaceCardGeneration::oop_since_save_marks_iterate_nv() (Perm 領域を処理)
                -> ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()
    
   (3) 以上の処理で見つかった参照オブジェクトを処理する.
       -> ReferenceProcessor::process_discovered_references()
          (なお使用するクロージャーは,
           DefNewGeneration::IsAliveClosure, DefNewGeneration::FastKeepAliveClosure, DefNewGeneration::FastEvacuateFollowersClosure.
          AbstractRefProcTaskExecutor は使用しない (NULL を渡す))
          -> (後述) (See: [here](no289169tf.html) for details)
    
   (4)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### DefNewGeneration::collect()
(#Under Construction)
See: [here](no28916XJg.html) for details
### GenCollectedHeap::gen_process_strong_roots()
(#Under Construction)
See: [here](no28916kTm.html) for details
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


```
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

### FastScanClosure::do_oop()
See: [here](no28916lbh.html) for details
### FastScanClosure::do_oop_work()
See: [here](no28916yln.html) for details
### OopsInGenClosure::do_barrier()
See: [here](no2114iQh.html) for details
### DefNewGeneration::copy_to_survivor_space()
See: [here](no28916_vt.html) for details
### ContiguousSpace::allocate()
See: [here](no21493ZD2.html) for details
### ContiguousSpace::allocate_impl()
See: [here](no21493lhR.html) for details
### Generation::promote()
See: [here](no28916M6z.html) for details
### OneContigSpaceCardGeneration::allocate()
See: [here](no21493wbh.html) for details

### DefNewGeneration::FastEvacuateFollowersClosure::do_void()
See: [here](no28916xds.html) for details
### GenCollectedHeap::oop_since_save_marks_iterate()
See: [here](no28916-ny.html) for details
### DefNewGeneration::oop_since_save_marks_iterate_v()  または  DefNewGeneration::oop_since_save_marks_iterate_nv()
(#Under Construction)
See: [here](no28916KGO.html) for details
### ContiguousSpace::oop_since_save_marks_iterate_v()  または  ContiguousSpace::oop_since_save_marks_iterate_nv()
(#Under Construction)
See: [here](no28916XQU.html) for details
### oopDesc::oop_iterate(OopClosureType* blk)
See: [here](no28916-um.html) for details
### oopDesc::oop_iterate(OopClosureType* blk, MemRegion mr)
See: [here](no28916L5s.html) for details
### *Klass::oop_oop_iterate() または *Klass::oop_oop_iterate_v() または *Klass::oop_oop_iterate_nv()
(#Under Construction)


### DefNewGeneration::swap_spaces()
(#Under Construction)

### DefNewGeneration::remove_forwarding_pointers()
(#Under Construction)







