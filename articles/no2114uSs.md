---
layout: default
title: Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： ParallelScavengeHeap 用の初期化処理の流れ 
---
[Up](noS8y7MAwP.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： ParallelScavengeHeap 用の初期化処理の流れ 

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: [here](noYV_1Xq7P.html) for details)
-> ParallelScavengeHeap::initialize()
   -> (1) 初期化処理の前準備を行う
          -> CollectedHeap::pre_initialize()

      (1) 各世代(Young/Old/Perm)の領域サイズを決定する
          -> GenerationSizer::GenerationSizer()
             -> (1) スーパークラスのコンストラクタの呼び出し
                    -> CollectorPolicy::CollectorPolicy()

                (2) ヒープサイズに関するコマンドラインオプション(-Xms,-Xmx等)の値を取得する
                    -> GenerationSizer::initialize_flags()
                       -> TwoGenerationCollectorPolicy::initialize_flags()
                          -> GenCollectorPolicy::initialize_flags()
                             -> CollectorPolicy::initialize_flags()
                (3) 各世代(Young/Old/Perm)の領域サイズを決定する
                    -> TwoGenerationCollectorPolicy::initialize_size_info()
                       -> GenCollectorPolicy::initialize_size_info()
                          -> CollectorPolicy::initialize_size_info()

      (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する
          -> Universe::preferred_heap_base()

      (1) ヒープ領域をメモリ空間上に reserve する (まず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保)
          -> ReservedHeapSpace::ReservedHeapSpace(const size_t prefix_size, const size_t prefix_align, const size_t suffix_size, const size_t suffix_align, char* requested_address)
             -> ReservedSpace::ReservedSpace(const size_t prefix_size, const size_t prefix_align, const size_t suffix_size, const size_t suffix_align, char* requested_address, const size_t noaccess_prefix)
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
                (1) 確保した領域のアラインメントが条件に合っていなければ確保し直す.
                -> ReservedSpace::reserve_and_align()
                   -> os::reserve_memory()
                      -> (同上)
                   -> ReservedSpace::align_reserved_region()
             -> ReservedSpace::protect_noaccess_prefix()
                -> os::protect_memory()               (← UseCompressedOops の場合のみ実行)

      (1) 確保したヒープ領域に対応する Barrier Set (CardTableExtension オブジェクト) を生成
          -> CardTableExtension::CardTableExtension()

      (1) 上で1つの連続領域として確保したヒープ空間を Perm とそれ以外に分ける
      (1) さらに, Perm 以外の領域を Old と Young に分け, 対応する Generation オブジェクトを生成 (PSYoungGen, PSOldGen)
          (ついでに, ここまでは reserve しただけだった メモリ領域の commit も行う)
          -> AdjoiningGenerations::AdjoiningGenerations()
             -> AdjoiningVirtualSpaces::AdjoiningVirtualSpaces()
             -> UseAdaptiveGCBoundary オプションの値に応じて処理が分岐
                * UseAdaptiveGCBoundary が true の場合:
                  -> AdjoiningVirtualSpaces::initialize()
                     -> PSVirtualSpace::expand_by()
                        -> os::commit_memory()
                     -> PSVirtualSpaceHighToLow::expand_by()
                        -> os::commit_memory()
                  -> ASPSYoungGen::ASPSYoungGen()
                  -> ASPSOldGen::ASPSOldGen()
                  -> PSYoungGen::initialize_work()
                  -> PSOldGen::initialize_work()
                * UseAdaptiveGCBoundary が false の場合:
                  -> PSYoungGen::PSYoungGen()
                  -> PSOldGen::PSOldGen()
                  -> PSYoungGen::initialize()
                  -> PSOldGen::initialize()

      (1) ParallelScavengeHeap 用の AdaptiveSizePolicy (PSAdaptiveSizePolicy オブジェクト) を作成
          -> PSAdaptiveSizePolicy::PSAdaptiveSizePolicy()

      (1) Perm についても Generation オブジェクトを作成 (PSPermGen)
          -> PSPermGen::PSPermGen()

      (1) ParallelScavengeHeap 用の GCAdaptivePolicyCounters (PSGCAdaptivePolicyCounters オブジェクト) を作成
          -> PSGCAdaptivePolicyCounters::PSGCAdaptivePolicyCounters()

      (1) 並列 GC 用の作業スレッドを生成する.
          -> GCTaskManager::create()
             -> (See: [here](no24805iK.html) for details)

      (1) UseParallelOldGC オプションが指定されていれば, そのための初期化処理を行う
          -> PSParallelCompact::initialize()
             -> ParallelCompactData::initialize()
                -> ParallelCompactData::initialize_region_data()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ParallelScavengeHeap::initialize()
See: [here](no344Yjc.html) for details
### CollectedHeap::pre_initialize()
See: [here](no344xEK.html) for details

### GenerationSizer::GenerationSizer()
See: [here](no344YqQ.html) for details
### CollectorPolicy::CollectorPolicy()
See: [here](no31977fVf.html) for details
### GenerationSizer::initialize_flags()
See: [here](no344y-c.html) for details
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

### Universe::preferred_heap_base()
(#Under Construction)

### ReservedHeapSpace::ReservedHeapSpace(const size_t prefix_size, const size_t prefix_align, const size_t suffix_size, const size_t suffix_align, char* requested_address)
See: [here](no344Nbk.html) for details
### ReservedSpace::ReservedSpace(const size_t prefix_size, const size_t prefix_align, const size_t suffix_size, const size_t suffix_align, char* requested_address, const size_t noaccess_prefix)
See: [here](no344052.html) for details
### ReservedSpace::initialize()
See: [here](no344z4v.html) for details
### os::reserve_memory_special()
(#Under Construction)

### os::attempt_reserve_memory_at()
(#Under Construction)

### ReservedSpace::reserve_and_align()
See: [here](no344mDG.html) for details
### ReservedSpace::align_reserved_region()
See: [here](no344zNM.html) for details
### ReservedSpace::protect_noaccess_prefix()
See: [here](no344nvw.html) for details
### os::protect_memory()
(#Under Construction)

### CardTableExtension::CardTableExtension()
(#Under Construction)

### AdjoiningGenerations::AdjoiningGenerations()
See: [here](no344AYS.html) for details
### AdjoiningVirtualSpaces::AdjoiningVirtualSpaces()
See: [here](no344ase.html) for details
### AdjoiningVirtualSpaces::initialize()
See: [here](no344n2k.html) for details
### PSVirtualSpace::expand_by()
See: [here](no3440Ar.html) for details
### os::commit_memory()
(#Under Construction)

### PSVirtualSpaceHighToLow::expand_by()
See: [here](no344BLx.html) for details
### ASPSYoungGen::ASPSYoungGen(PSVirtualSpace* vs, size_t initial_byte_size, size_t minimum_byte_size, size_t byte_size_limit)
See: [here](no31977eiA.html) for details
### PSYoungGen::PSYoungGen()
See: [here](no31977rsG.html) for details
### ASPSOldGen::ASPSOldGen()
See: [here](no3197742M.html) for details
### PSOldGen::PSOldGen(size_t initial_size, size_t min_size, size_t max_size, const char* perf_data_name, int level)
See: [here](no31977FBT.html) for details
### PSYoungGen::initialize_work()
(#Under Construction)

### PSOldGen::initialize_work()
(#Under Construction)

### PSAdaptiveSizePolicy::PSAdaptiveSizePolicy()
(#Under Construction)

### PSPermGen::PSPermGen()
(#Under Construction)

### PSGCAdaptivePolicyCounters::PSGCAdaptivePolicyCounters()
(#Under Construction)

### PSParallelCompact::initialize()
See: [here](no3172wN1.html) for details
### ParallelCompactData::initialize()
See: [here](no3172iXE.html) for details
### ParallelCompactData::initialize_region_data()
See: [here](no3172vhK.html) for details






