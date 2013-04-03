---
layout: default
title: Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： ConcurrentG1RefineThread による remembered set の管理処理  
---
[Up](noPY0OPTFc.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： Garbage Collection を補佐する処理 ： ConcurrentG1RefineThread による remembered set の管理処理  

--- 
#Under Construction

## 概要(Summary)
ConcurrentG1RefineThread は, HotSpot の初期化時に G1CollectedHeap::initialize() の中で生成される.

ConcurrentG1RefineThread の作成は os::create_thread() 及び os::start_thread() で行われる.
このため, 生成された ConcurrentG1RefineThread スレッドは java_start() から実行が開始される (See: [here](noYHbL-pQM.html) for details).

java_start() から始まるスレッドの起動処理では最終的に Thread::run() が呼び出される (See: [here](no3059-9C.html) for details).
ConcurrentG1RefineThread は Thread::run() をオーバーライドしているので, 
実際に呼び出されるのは ConcurrentG1RefineThread::run() になる.
この ConcurrentG1RefineThread::run() の中で実際の remembered set の管理処理が行われる.

ConcurrentG1RefineThread は, HotSpot の初期化時に最大数分だけ生成された後, 仕事の量に応じて動的に稼働数を変える.
これは, 仕事のない ConcurrentG1RefineThread が自発的にブロックすることで実現されている.
一旦ブロックした ConcurrentG1RefineThread は, 仕事量が増加してくると他の ConcurrentG1RefineThread から, 
もしくは Mutator である JavaThread の Write barrier 処理から起床される.

(さらに, 仕事量が多すぎて ConcurrentG1RefineThread だけでは間に合わない場合, 
Mutator である通常の JavaThread も処理に加わる) (See: [here](no2114EV0.html) for details)

稼働数を管理するために 3つの閾値が用意されている("green", "yellow", "red"). 処理は以下のようになる.

1. 未処理の card 数が [0, green) の間は, card の処理は行わない
   (全ての ConcurrentG1RefineThread がブロックした状態).
2. 未処理の card 数が green 閾値を超え, [green, yellow) の間は, 
   徐々に ConcurrentG1RefineThread の稼働数を上げて card の処理を行う.
3. 未処理の card 数が [yellow, red) の間は, 
   全ての ConcurrentG1RefineThread が card の処理を行う.
4. 未処理の card 数が red を超えた場合, 
   Mutator である通常の JavaThread も card の処理を行う. 
   (PtrQueueSet::process_or_enqueue_complete_buffer() 参照 (See: [here](no2114EV0.html) for details))


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentG1Refine.hpp))
     /*
      * The value of the update buffer queue length falls into one of 3 zones:
      * green, yellow, red. If the value is in [0, green) nothing is
      * done, the buffers are left unprocessed to enable the caching effect of the
      * dirtied cards. In the yellow zone [green, yellow) the concurrent refinement
      * threads are gradually activated. In [yellow, red) all threads are
      * running. If the length becomes red (max queue length) the mutators start
      * processing the buffers.
      *
      * There are some interesting cases (when G1UseAdaptiveConcRefinement
      * is turned off):
      * 1) green = yellow = red = 0. In this case the mutator will process all
      *    buffers. Except for those that are created by the deferred updates
      *    machinery during a collection.
      * 2) green = 0. Means no caching. Can be a good way to minimize the
      *    amount of time spent updating rsets during a collection.
      */
      int _green_zone;
      int _yellow_zone;
      int _red_zone;
```

## 備考(Notes)
なお, ConcurrentG1RefineThread の稼働数の増減を実現するため,
各 ConcurrentG1RefineThread はそれぞれ個別の Monitor オブジェクトを保持している
(0 番目の ConcurrentG1RefineThread だけは, DirtyCardQ_CBL_mon を使用する).
各 ConcurrentG1RefineThread は, worker id が一つ小さい ConcurrentG1RefineThread によって起床される
(0 番目の ConcurrentG1RefineThread だけは, write barrier 処理から起床される). 
(See: ConcurrentG1RefineThread) (See: [here](no2114EV0.html) for details)


## 処理の流れ (概要)(Execution Flows : Summary)
### ConcurrentG1RefineThread を作成する処理
```
G1CollectedHeap::initialize() (See: [here](no2114tfN.html) for details)
-> ConcurrentG1Refine::ConcurrentG1Refine()
   -> ConcurrentG1RefineThread::ConcurrentG1RefineThread()
      -> ConcurrentG1RefineThread::initialize()
      -> ConcurrentGCThread::create_and_start()
         -> os::create_thread()
            -> (See: [here](noYHbL-pQM.html) for details)
         -> os::set_priority()
         -> os::start_thread()
            -> (See: [here](noYHbL-pQM.html) for details)
```

### 生成された ConcurrentG1RefineThread 側の処理
```
-> java_start()
   -> (See: [here](noaGdrH-zs.html), [here](noQiWP6ip-.html) and [here](nobwSeebST.html) for details)
      -> ConcurrentG1RefineThread::run()
         -> (1) 初期化
                -> ConcurrentGCThread::initialize_in_thread()

            (2) HotSpot の初期化が終わるまで待機する
                -> ConcurrentGCThread::wait_for_universe_init()

            (3) HotSpot が終了するまで, 以下の処理を無限ループ
                (1) 仕事がくるまで待機 (= Write Barrier 処理, もしくは他の ConcurrentG1RefineThread から起こされるまで待つ) (See: [here](no2114EV0.html) for details, ConcurrentG1RefineThread::run())
                    -> ConcurrentG1RefineThread::wait_for_completed_buffers()

                (1) 処理量に応じて, ConcurrentG1RefineThread の稼働数を増減させる
                    * 処理量が少ない場合:
                      -> ConcurrentG1RefineThread::deactivate()
                    * 処理量が多い場合:
                      -> ConcurrentG1RefineThread::activate() (<= 他の ConcurrentG1RefineThread を起こす処理)

                (1) card の処理を行う
                    -> DirtyCardQueueSet::apply_closure_to_completed_buffer(int worker_i, int stop_at, bool during_pause)
                       -> DirtyCardQueueSet::apply_closure_to_completed_buffer(CardTableEntryClosure* cl, int worker_i, int stop_at, bool during_pause)
                          -> DirtyCardQueueSet::get_completed_buffer()
                          -> DirtyCardQueueSet::apply_closure_to_completed_buffer_helper()
                             -> DirtyCardQueue::apply_closure_to_buffer()
                                -> RefineCardTableEntryClosure::do_card_ptr()
                                   -> G1RemSet::concurrentRefineOneCard()
                                      -> ConcurrentG1Refine::cache_insert()
                                      -> G1RemSet::concurrentRefineOneCard_impl()
                                         -> HeapRegion::oops_on_card_seq_iterate_careful()
                                            (この場合のクロージャーは, check_for_refs_into_cset 引数が false なので, UpdateRSOrPushRefOopClosure を FilterOutOfRegionClosure でくるんだもの)
                                            -> oopDesc::oop_iterate()
                                               -> FilterOutOfRegionClosure::do_oop()
                                                  -> FilterOutOfRegionClosure::do_oop_nv()
                                                     -> UpdateRSOrPushRefOopClosure::do_oop()
                                                        -> UpdateRSOrPushRefOopClosure::do_oop_work()
                                                           -> G1RemSet::par_write_ref()
                                                              -> HeapRegionRemSet::add_reference(OopOrNarrowOopStar from, int tid)
                                                                 -> OtherRegionsTable::add_reference()
                                                                    -> OtherRegionsTable::find_region_table()
                                                                    -> 新しいエントリが必要なら以下の処理を行う.
                                                                       -> SparsePRT::add_card() (<= これは, 成功すればここでリターン)
                                                                          -> RSHashTable::add_card()
                                                                             -> RSHashTable::entry_for_region_ind_create()
                                                                             -> SparsePRTEntry::add_card()
                                                                       -> * もういっぱいなので coarse map に追い出して確保する場合
                                                                            -> OtherRegionsTable::delete_region_table()
                                                                          * まだ空きがある場合
                                                                            -> PosParPRT::alloc()
                                                                    -> PosParPRT::add_reference()
                                                                       -> PerRegionTable::add_reference()
                                                                          -> PerRegionTable::add_reference_work()
                                                                             -> PerRegionTable::add_card_work()
                                                                                -> * 
                                                                                     -> BitMap::par_at_put()
                                                                                   * 
                                                                                     -> BitMap::at_put()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ConcurrentG1RefineThread::ConcurrentG1RefineThread()
See: [here](no2935Ptj.html) for details
### ConcurrentGCThread::ConcurrentGCThread()
(#Under Construction)

### ConcurrentG1RefineThread::initialize()
See: [here](no29352L2.html) for details
### ConcurrentGCThread::create_and_start()
See: [here](no2935pBw.html) for details

### ConcurrentG1RefineThread::run()
See: [here](no2935oOR.html) for details

### ConcurrentG1RefineThread::run_young_rs_sampling()
See: [here](no3127fWB.html) for details
### ConcurrentG1RefineThread::sample_young_list_rs_lengths()
See: [here](no3127sgH.html) for details

### ConcurrentG1RefineThread::wait_for_completed_buffers()
See: [here](no29351YX.html) for details
### ConcurrentG1RefineThread::is_active()
See: [here](no2935Cjd.html) for details
### ConcurrentG1RefineThread::activate()
See: [here](no2935c3p.html) for details
### ConcurrentG1RefineThread::deactivate()
See: [here](no2935P0X.html) for details
### DirtyCardQueueSet::apply_closure_to_completed_buffer(int worker_i, int stop_at, bool during_pause)
See: [here](no2935c-d.html) for details
### DirtyCardQueueSet::apply_closure_to_completed_buffer(CardTableEntryClosure* cl, int worker_i, int stop_at, bool during_pause)
See: [here](no2935pIk.html) for details
### DirtyCardQueueSet::get_completed_buffer()
See: [here](no29352Sq.html) for details
### DirtyCardQueueSet::apply_closure_to_completed_buffer_helper()
See: [here](no2935Ddw.html) for details
### DirtyCardQueue::apply_closure_to_buffer()
See: [here](no2935Qn2.html) for details
### RefineCardTableEntryClosure::do_card_ptr()
See: [here](no31275qN.html) for details
### G1RemSet::concurrentRefineOneCard()
See: [here](no3127G1T.html) for details
##### 備考(Notes)
(なお, 引数などの意味は以下を参照)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1RemSet.hpp))
      // Refine the card corresponding to "card_ptr".  If "sts" is non-NULL,
      // join and leave around parts that must be atomic wrt GC.  (NULL means
      // being done at a safepoint.)
      // If check_for_refs_into_cset is true, a true result is returned
      // if the given card contains oops that have references into the
      // current collection set.
```

### G1RemSet::concurrentRefineOneCard_impl()
See: [here](no3127gJg.html) for details
### HeapRegion::oops_on_card_seq_iterate_careful()
See: [here](no3127tTm.html) for details
### FilterOutOfRegionClosure::do_oop()

```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.hpp))
      virtual void do_oop(oop* p) { do_oop_nv(p); }
      virtual void do_oop(narrowOop* p) { do_oop_nv(p); }
```

### FilterOutOfRegionClosure::do_oop_nv()
See: [here](no3420hQK.html) for details
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








