---
layout: default
title: Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： GenCollectedHeap 用の初期化処理の流れ 
---
[Up](noS8y7MAwP.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： GenCollectedHeap 用の初期化処理の流れ 

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### CollectorPolicy の初期化
#### 非 CMS 用
```
(See: [here](noYV_1Xq7P.html) for details)
-> MarkSweepPolicy::MarkSweepPolicy()
   -> (1) スーパークラスのコンストラクタの呼び出し
          -> CollectorPolicy::CollectorPolicy()

      (1) 各世代(Young/Old/Perm)の領域サイズを決定し, 対応する GenerationSpec オブジェクトを作成する.
          -> GenCollectorPolicy::initialize_all()
             -> (1) ヒープサイズに関するコマンドラインオプション(-Xms,-Xmx等)の値を取得する
                    -> TwoGenerationCollectorPolicy::initialize_flags()
                       -> GenCollectorPolicy::initialize_flags()
                          -> CollectorPolicy::initialize_flags()

                (2) 各世代(Young/Old/Perm)の領域サイズを決定する
                    -> TwoGenerationCollectorPolicy::initialize_size_info()
                       -> GenCollectorPolicy::initialize_size_info()
                          -> CollectorPolicy::initialize_size_info()

                (3) 各世代に対応する GenerationSpec オブジェクトを作成する
                    -> MarkSweepPolicy::initialize_generations()
```

#### CMS 用
```
(See: [here](noYV_1Xq7P.html) for details)
-> ConcurrentMarkSweepPolicy::ConcurrentMarkSweepPolicy()
   -> GenCollectorPolicy::initialize_all()
      -> (同上)

(See: [here](noYV_1Xq7P.html) for details)
-> ASConcurrentMarkSweepPolicy::ASConcurrentMarkSweepPolicy()
   -> 特にオーバーライドされていないので, ConcurrentMarkSweepPolicy::ConcurrentMarkSweepPolicy() と同じ
      -> (同上)

```

### CollectedHeap の初期化
```
(See: [here](noYV_1Xq7P.html) for details)
-> GenCollectedHeap::initialize()
   -> (1) 初期化処理の前準備を行う
          -> CollectedHeap::pre_initialize()

      (1) ヒープ領域をメモリ空間上に reserve する (まず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保)
          -> GenCollectedHeap::allocate()
             -> (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する
                    -> Universe::preferred_heap_base()

                (2) ヒープ領域をメモリ空間上に reserve する
                    -> ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t forced_base_alignment, bool large, char* requested_address)
                       -> ReservedSpace::ReservedSpace()
                          -> ReservedSpace::initialize()
                             -> (1) 以下のどれかでメモリ領域を reserve する.
                                    * Large Page を使用したいが, OS の制約により large page については
                                      reserve と commit は同時に行わなくてはいけない場合:
                                      -> os::reserve_memory_special()
                                         -> shmat(), VirtualAlloc(), etc
                                            (各 OS 固有の large page なメモリ空間確保用のシステムコールを呼び出す)
                                    * 確保するアドレスが指定されている場合:
                                      -> os::attempt_reserve_memory_at()
                                         -> os::reserve_memory()
                                            -> mmap(), VirtualAlloc(), etc
                                               (各 OS 固有の仮想メモリ空間確保用のシステムコール)
                                    * それ以外の場合:
                                      -> os::reserve_memory()
                                         -> (同上)
                       -> ReservedSpace::protect_noaccess_prefix()
                          -> os::protect_memory()          (← UseCompressedOops の場合のみ実行)

      (1) 確保したヒープ領域に対応する Remembered Set (GenRemSet オブジェクト) を生成
          -> CollectorPolicy::create_rem_set()

      (1) 1つの連続領域として確保したヒープ空間を各世代(Young/Old/Perm)に分け,
          New/Old に対応する Generation オブジェクトを生成.
          (あわせて, ここまでは reserve しただけだった メモリ領域の commit も行う)

          -> GenerationSpec::init()
             -> 各Generationクラスのコンストラクタを呼び出す. 呼び出し先はGenerationクラスに応じて異なる.

                * Generation::DefNew の場合:
                  -> DefNewGeneration::DefNewGeneration()
                     -> Generation::Generation()
                        -> VirtualSpace::initialize()
                           -> VirtualSpace::expand_by()
                              -> os::commit_memory()

                * Generation::MarkSweepCompact の場合:
                  -> TenuredGeneration::TenuredGeneration()
                     -> OneContigSpaceCardGeneration::OneContigSpaceCardGeneration()
                        -> CardGeneration::CardGeneration()
                           -> Generation::Generation()
                              -> (同上)

                * Generation::ParNew の場合:
                  -> ParNewGeneration::ParNewGeneration()
                     -> DefNewGeneration::DefNewGeneration()
                        -> (同上)

                * Generation::ASParNew の場合:
                  -> ASParNewGeneration::ASParNewGeneration()
                     -> ParNewGeneration::ParNewGeneration()
                        -> (同上)

                * Generation::ConcurrentMarkSweep の場合:
                  -> ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration()
                     -> CardGeneration::CardGeneration()
                        -> Generation::Generation()
                           -> (同上)
                  -> ConcurrentMarkSweepGeneration::initialize_performance_counters()

                * Generation::ASConcurrentMarkSweep の場合:
                  -> ASConcurrentMarkSweepGeneration::ASConcurrentMarkSweepGeneration()
                     -> ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration()
                        -> (同上)
                  -> ConcurrentMarkSweepGeneration::initialize_performance_counters()

      (1) Perm に対応する Generation オブジェクトを生成
          -> PermanentGenerationSpec::init()
             -> 各Generationクラスのコンストラクタを呼び出す. 呼び出し先はGenerationクラスに応じて異なる.
                * PermGen::MarkSweepCompact の場合:
                  -> CompactingPermGen::CompactingPermGen()
                     -> CompactingPermGenGen::CompactingPermGenGen()
                        -> OneContigSpaceCardGeneration::OneContigSpaceCardGeneration()
                           -> CardGeneration::CardGeneration()
                              -> Generation::Generation()
                                 -> (同上)
                     -> CompactingPermGenGen::initialize_performance_counters()

                * PermGen::ConcurrentMarkSweep の場合:
                  -> CMSPermGen::CMSPermGen()

      (1) 
          -> GenCollectedHeap::clear_incremental_collection_failed()

      (1) もし CMS であれば, GC 用のスレッドを作成する.
          -> GenCollectedHeap::create_cms_collector()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### MarkSweepPolicy::MarkSweepPolicy()
See: [here](no344MM1.html) for details

### GenCollectorPolicy::initialize_all()
See: [here](no344-VE.html) for details
### TwoGenerationCollectorPolicy::initialize_flags()
See: [here](no344LgK.html) for details
### GenCollectorPolicy::initialize_flags()
See: [here](no31977F6e.html) for details
### CollectorPolicy::initialize_flags()
See: [here](no31977SEl.html) for details
### TwoGenerationCollectorPolicy::initialize_size_info()
See: [here](no344MTp.html) for details
### GenCollectorPolicy::initialize_size_info()
See: [here](no31977fOr.html) for details
### CollectorPolicy::initialize_size_info()
See: [here](no31977sYx.html) for details
### MarkSweepPolicy::initialize_generations()
See: [here](no344_Ij.html) for details

### ConcurrentMarkSweepPolicy::ConcurrentMarkSweepPolicy()
See: [here](no3197753T.html) for details

### GenCollectedHeap::initialize()
See: [here](no344y3o.html) for details
### CollectedHeap::pre_initialize()
See: [here](no344xEK.html) for details
### GenCollectedHeap::allocate()
See: [here](no344_Bv.html) for details
### ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t forced_base_alignment, bool large, char* requested_address)
See: [here](no344Zkj.html) for details
### ReservedSpace::ReservedSpace(size_t size, size_t alignment, bool large, char* requested_address, const size_t noaccess_prefix)
See: [here](no31977gPy.html) for details
### ReservedSpace::initialize()
See: [here](no344z4v.html) for details
### ReservedSpace::protect_noaccess_prefix()
See: [here](no344nvw.html) for details
### os::protect_memory()
(#Under Construction)

### CollectorPolicy::create_rem_set()
(#Under Construction)

### GenerationSpec::init()
See: [here](no344ZyL.html) for details
### DefNewGeneration::DefNewGeneration()
See: [here](no31977GJO.html) for details
### Generation::Generation()
See: [here](no31977TMg.html) for details
### VirtualSpace::initialize()
See: [here](no31977gWm.html) for details
### VirtualSpace::expand_by()
See: [here](no31977tgs.html) for details
### os::commit_memory()
(#Under Construction)

### TenuredGeneration::TenuredGeneration()
See: [here](no319775-H.html) for details
### OneContigSpaceCardGeneration::OneContigSpaceCardGeneration()
See: [here](no319776qy.html) for details
### CardGeneration::CardGeneration()
See: [here](no31977s0B.html) for details(#Under Construction)

### ParNewGeneration::ParNewGeneration()
(#Under Construction)

### ASParNewGeneration::ASParNewGeneration()
(#Under Construction)

### ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration()
(#Under Construction)

### ConcurrentMarkSweepGeneration::initialize_performance_counters()
(#Under Construction)

### ASConcurrentMarkSweepGeneration::ASConcurrentMarkSweepGeneration()
(#Under Construction)

### PermanentGenerationSpec::init()
See: [here](no78821XK.html) for details
### CompactingPermGen::CompactingPermGen()
See: [here](no31977TTU.html) for details
### CompactingPermGenGen::CompactingPermGenGen()
See: [here](no31977tng.html) for details
### CompactingPermGenGen::initialize_performance_counters()
See: [here](no31977gda.html) for details
### CMSPermGen::CMSPermGen()
(#Under Construction)

### GenCollectedHeap::clear_incremental_collection_failed()
See: [here](no31977GCa.html) for details
### GenCollectedHeap::create_cms_collector()
(#Under Construction)







