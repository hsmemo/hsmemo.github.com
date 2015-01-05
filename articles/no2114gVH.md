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
<div class="flow-abst"><pre>
(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; MarkSweepPolicy::MarkSweepPolicy()
   -&gt; (1) スーパークラスのコンストラクタの呼び出し
          -&gt; CollectorPolicy::CollectorPolicy()

      (1) 各世代(Young/Old/Perm)の領域サイズを決定し, 対応する GenerationSpec オブジェクトを作成する.
          -&gt; GenCollectorPolicy::initialize_all()
             -&gt; (1) ヒープサイズに関するコマンドラインオプション(-Xms,-Xmx等)の値を取得する
                    -&gt; TwoGenerationCollectorPolicy::initialize_flags()
                       -&gt; GenCollectorPolicy::initialize_flags()
                          -&gt; CollectorPolicy::initialize_flags()

                (2) 各世代(Young/Old/Perm)の領域サイズを決定する
                    -&gt; TwoGenerationCollectorPolicy::initialize_size_info()
                       -&gt; GenCollectorPolicy::initialize_size_info()
                          -&gt; CollectorPolicy::initialize_size_info()

                (3) 各世代に対応する GenerationSpec オブジェクトを作成する
                    -&gt; MarkSweepPolicy::initialize_generations()
</pre></div>

#### CMS 用
<div class="flow-abst"><pre>
(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; ConcurrentMarkSweepPolicy::ConcurrentMarkSweepPolicy()
   -&gt; GenCollectorPolicy::initialize_all()
      -&gt; (同上)

(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; ASConcurrentMarkSweepPolicy::ASConcurrentMarkSweepPolicy()
   -&gt; 特にオーバーライドされていないので, ConcurrentMarkSweepPolicy::ConcurrentMarkSweepPolicy() と同じ
      -&gt; (同上)

</pre></div>

### CollectedHeap の初期化
<div class="flow-abst"><pre>
(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; GenCollectedHeap::initialize()
   -&gt; (1) 初期化処理の前準備を行う
          -&gt; CollectedHeap::pre_initialize()

      (1) ヒープ領域をメモリ空間上に reserve する (まず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保)
          -&gt; GenCollectedHeap::allocate()
             -&gt; (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する
                    -&gt; Universe::preferred_heap_base()

                (2) ヒープ領域をメモリ空間上に reserve する
                    -&gt; ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t forced_base_alignment, bool large, char* requested_address)
                       -&gt; ReservedSpace::ReservedSpace()
                          -&gt; ReservedSpace::initialize()
                             -&gt; (1) 以下のどれかでメモリ領域を reserve する.
                                    * Large Page を使用したいが, OS の制約により large page については
                                      reserve と commit は同時に行わなくてはいけない場合:
                                      -&gt; os::reserve_memory_special()
                                         -&gt; shmat(), VirtualAlloc(), etc
                                            (各 OS 固有の large page なメモリ空間確保用のシステムコールを呼び出す)
                                    * 確保するアドレスが指定されている場合:
                                      -&gt; os::attempt_reserve_memory_at()
                                         -&gt; os::reserve_memory()
                                            -&gt; mmap(), VirtualAlloc(), etc
                                               (各 OS 固有の仮想メモリ空間確保用のシステムコール)
                                    * それ以外の場合:
                                      -&gt; os::reserve_memory()
                                         -&gt; (同上)
                       -&gt; ReservedSpace::protect_noaccess_prefix()
                          -&gt; os::protect_memory()          (← UseCompressedOops の場合のみ実行)

      (1) 確保したヒープ領域に対応する Remembered Set (GenRemSet オブジェクト) を生成
          -&gt; CollectorPolicy::create_rem_set()

      (1) 1つの連続領域として確保したヒープ空間を各世代(Young/Old/Perm)に分け,
          New/Old に対応する Generation オブジェクトを生成.
          (あわせて, ここまでは reserve しただけだった メモリ領域の commit も行う)

          -&gt; GenerationSpec::init()
             -&gt; 各Generationクラスのコンストラクタを呼び出す. 呼び出し先はGenerationクラスに応じて異なる.

                * Generation::DefNew の場合:
                  -&gt; DefNewGeneration::DefNewGeneration()
                     -&gt; Generation::Generation()
                        -&gt; VirtualSpace::initialize()
                           -&gt; VirtualSpace::expand_by()
                              -&gt; os::commit_memory()

                * Generation::MarkSweepCompact の場合:
                  -&gt; TenuredGeneration::TenuredGeneration()
                     -&gt; OneContigSpaceCardGeneration::OneContigSpaceCardGeneration()
                        -&gt; CardGeneration::CardGeneration()
                           -&gt; Generation::Generation()
                              -&gt; (同上)

                * Generation::ParNew の場合:
                  -&gt; ParNewGeneration::ParNewGeneration()
                     -&gt; DefNewGeneration::DefNewGeneration()
                        -&gt; (同上)

                * Generation::ASParNew の場合:
                  -&gt; ASParNewGeneration::ASParNewGeneration()
                     -&gt; ParNewGeneration::ParNewGeneration()
                        -&gt; (同上)

                * Generation::ConcurrentMarkSweep の場合:
                  -&gt; ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration()
                     -&gt; CardGeneration::CardGeneration()
                        -&gt; Generation::Generation()
                           -&gt; (同上)
                  -&gt; ConcurrentMarkSweepGeneration::initialize_performance_counters()

                * Generation::ASConcurrentMarkSweep の場合:
                  -&gt; ASConcurrentMarkSweepGeneration::ASConcurrentMarkSweepGeneration()
                     -&gt; ConcurrentMarkSweepGeneration::ConcurrentMarkSweepGeneration()
                        -&gt; (同上)
                  -&gt; ConcurrentMarkSweepGeneration::initialize_performance_counters()

      (1) Perm に対応する Generation オブジェクトを生成
          -&gt; PermanentGenerationSpec::init()
             -&gt; 各Generationクラスのコンストラクタを呼び出す. 呼び出し先はGenerationクラスに応じて異なる.
                * PermGen::MarkSweepCompact の場合:
                  -&gt; CompactingPermGen::CompactingPermGen()
                     -&gt; CompactingPermGenGen::CompactingPermGenGen()
                        -&gt; OneContigSpaceCardGeneration::OneContigSpaceCardGeneration()
                           -&gt; CardGeneration::CardGeneration()
                              -&gt; Generation::Generation()
                                 -&gt; (同上)
                     -&gt; CompactingPermGenGen::initialize_performance_counters()

                * PermGen::ConcurrentMarkSweep の場合:
                  -&gt; CMSPermGen::CMSPermGen()

      (1) 
          -&gt; GenCollectedHeap::clear_incremental_collection_failed()

      (1) もし CMS であれば, GC 用のスレッドを作成する.
          -&gt; GenCollectedHeap::create_cms_collector()
</pre></div>


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







