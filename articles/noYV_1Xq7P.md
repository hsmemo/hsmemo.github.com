---
layout: default
title: Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (1)  
---
[Up](no2114hIm.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (1)  

--- 
## 概要(Summary)
メモリ関係のデータ構造は HotSpot の起動時に初期化される.
この初期化時には, まず使用する GC アルゴリズムが決定される.

* コマンドラインオプションで明示的に指定された場合, 指定されたものが使用される.
* 明示的な指定がない場合, Arguments::set_ergonomics_flags() 内で Ergonomics により決定される.

その後, Universe::initialize_heap() の中で
実際のヒープ領域の確保やその管理用のオブジェクトの生成が行われる
(See: [here](noS8y7MAwP.html) and [here](no7882OK1.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> (1) コマンドラインオプションに基づき GC アルゴリズムを決定する. 指定がなければ適当なものを選択する.
          -> Arguments::parse()
             -> Arguments::set_ergonomics_flags()
                -> os::is_server_class_machine()
                -> Arguments::should_auto_select_low_pause_collector()

      (2) GC アルゴリズムに応じた CollectorPolicy オブジェクトおよび CollectedHeap オブジェクトを生成＆初期化する
          -> init_globals()
             -> universe_init()
                -> Universe::initialize_heap()
                   -> (1) コマンドラインオプションに応じて,
                          適切な CollectorPolicy およびヒープ領域管理用のクラス(CollectedHeap のサブクラス)を生成する.

                          * UseParallelGC の場合:
                            * CollectorPolicy : 無し 
                              (正確には GenerationSizer が CollectorPolicy に相当するが, 後で生成される)
                            * CollectedHeap   : ParallelScavengeHeap()
                          * UseG1GC の場合:
                            * CollectorPolicy : G1CollectorPolicy_BestRegionsFirst()
                            * CollectedHeap   : G1CollectedHeap()
                          * UseSerialGC の場合
                            * CollectorPolicy : MarkSweepPolicy()
                            * CollectedHeap   : GenCollectedHeap()
                          * UseConcMarkSweepGC && UseAdaptiveSizePolicy の場合
                            * CollectorPolicy : ASConcurrentMarkSweepPolicy()
                            * CollectedHeap   : GenCollectedHeap()
                          * UseConcMarkSweepGC && ! UseAdaptiveSizePolicy の場合
                            * CollectorPolicy : ConcurrentMarkSweepPolicy()  
                            * CollectedHeap   : GenCollectedHeap()
                          * それ以外の場合
                            * CollectorPolicy : MarkSweepPolicy()
                            * CollectedHeap   : GenCollectedHeap()

                      (2) 生成した CollectedHeap オブジェクトの CollectedHeap::initialize() メソッドを呼び出す.
                          (これは各ヒープクラスごとに以下のようにオーバーライドされている.
                          この中で, 使用するGCアルゴリズムに合わせたヒープ領域の確保が行われる)

                          * UseParallelGCParallelScavenge の場合:

                            -> ParallelScavengeHeap::initialize()
                               -> (See: [here](no2114uSs.html) for details)

                          * UseG1GC の場合:

                            -> G1CollectedHeap::initialize()
                               -> (See: [here](no2114tfN.html) for details)

                          * それ以外の場合:

                            -> GenCollectedHeap::initialize()
                               -> (See: [here](no2114gVH.html) for details)

                      (3) UseCompressedOops 使用時には, ヒープのベースアドレスとポインタのシフト幅を設定しておく
                          -> Universe::set_narrow_oop_base()
                          -> Universe::set_narrow_oop_shift()

                      (4) UseTLAB オプションが指定されていれば ThreadLocalAllocBuffer の初期化を行う
                          -> ThreadLocalAllocBuffer::startup_initialization()

      (3) GC アルゴリズムとして CMS か G1GC が指定されている場合には, SurrogateLockerThread を生成する
          -> ConcurrentMarkSweepThread::makeSurrogateLockerThread()  or  ConcurrentMarkThread::makeSurrogateLockerThread()
             -> (See: [here](no7882OK1.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### Arguments::set_ergonomics_flags()
See: [here](no344lmu.html) for details
### os::is_server_class_machine()
See: [here](no319774vY.html) for details
### Arguments::should_auto_select_low_pause_collector()
See: [here](no31977rlS.html) for details

### universe_init()
See: [here](no344yw0.html) for details
### Universe::initialize_heap()
See: [here](no344k6D.html) for details
### ThreadLocalAllocBuffer::startup_initialization()
See: [here](no344-OQ.html) for details






