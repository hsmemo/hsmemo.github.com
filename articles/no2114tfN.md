---
layout: default
title: Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： G1CollectedHeap 用の初期化処理の流れ 
---
[Up](noS8y7MAwP.html) [Top](../index.html)

#### Memory allocation (& GC 処理) ： メモリ関係の初期化処理の流れ (2) ： G1CollectedHeap 用の初期化処理の流れ 

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### CollectorPolicy の初期化
<div class="flow-abst"><pre>
(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; G1CollectorPolicy_BestRegionsFirst::G1CollectorPolicy_BestRegionsFirst()
   -&gt; (1) スーパークラスのコンストラクタの呼び出し
          -&gt; G1CollectorPolicy::G1CollectorPolicy()
             -&gt; CollectorPolicy::CollectorPolicy()
             -&gt; HeapRegion::setup_heap_region_size()
             -&gt; HeapRegionRemSet::setup_remset_size()
             -&gt; G1CollectorPolicy::initialize_all()
                -&gt; G1CollectorPolicy::initialize_flags()
                   -&gt; CollectorPolicy::initialize_flags()
                -&gt; CollectorPolicy::initialize_size_info()
                -&gt; CollectorPolicy::initialize_perm_generation()

      (2) CollectionSetChooser オブジェクトを生成する
          -&gt; CollectionSetChooser::CollectionSetChooser()
</pre></div>

### CollectedHeap の初期化
<div class="flow-abst"><pre>
(See: <a href="noYV_1Xq7P.html">here</a> for details)
-&gt; G1CollectedHeap::initialize()
   -&gt; (1) 初期化処理の前準備を行う
          -&gt; CollectedHeap::pre_initialize()
          -&gt; os::enable_vtime()
             -&gt; OS によって処理が異なる. ただし Solaris 以外の場合は何もしない.
                * Linux の場合:
                * Windows の場合:
                  -&gt; 何もしない
                * Solaris の場合:
                  -&gt; open(&quot;/proc/self/ctl&quot;, O_WRONLY)
                  -&gt; write(fd, { PCSET, PR_MSACCT }, ...)

      (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する
          -&gt; Universe::preferred_heap_base()

      (1) ヒープ領域をメモリ空間上に reserve する (まず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保)
          -&gt; ReservedSpace::ReservedSpace(size_t size, size_t alignment, bool large, char* requested_address = NULL, const size_t noaccess_prefix = 0)
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

      (1) 
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### G1CollectorPolicy_BestRegionsFirst::G1CollectorPolicy_BestRegionsFirst()
See: [here](no344ARe.html) for details
### G1CollectorPolicy::G1CollectorPolicy()
See: [here](no31977SLZ.html) for details
### CollectorPolicy::CollectorPolicy()
See: [here](no31977fVf.html) for details
### HeapRegion::setup_heap_region_size()
See: [here](no3197749A.html) for details
### HeapRegionRemSet::setup_remset_size()
See: [here](no31977FIH.html) for details
### G1CollectorPolicy::initialize_all()
See: [here](no319775pr.html) for details
### G1CollectorPolicy::initialize_flags()
See: [here](no31977G0x.html) for details
### CollectorPolicy::initialize_flags()
See: [here](no31977SEl.html) for details
### CollectorPolicy::initialize_size_info()
See: [here](no31977sYx.html) for details
### CollectorPolicy::initialize_perm_generation()
See: [here](no31977sfl.html) for details
### CollectionSetChooser::CollectionSetChooser()
See: [here](no31977SSN.html) for details

### G1CollectedHeap::initialize()
See: [here](no344zGY.html) for details
### CollectedHeap::pre_initialize()
See: [here](no344xEK.html) for details
### enable_vtime() (Linux の場合)
See: [here](no319775wf.html) for details
### enable_vtime() (Solaris の場合)
See: [here](no31977TFs.html) for details
### enable_vtime() (Windows の場合)
See: [here](no31977G7l.html) for details
### ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t forced_base_alignment, bool large, char* requested_address)
See: [here](no344052.html) for details
### ReservedSpace::ReservedSpace(size_t size, size_t alignment, bool large, char* requested_address, const size_t noaccess_prefix)
See: [here](no31977gPy.html) for details
### ReservedSpace::initialize()
See: [here](no344z4v.html) for details
### os::reserve_memory_special()
(#Under Construction)

### os::attempt_reserve_memory_at()
(#Under Construction)

### ReservedSpace::protect_noaccess_prefix()
See: [here](no344nvw.html) for details
### os::protect_memory()
(#Under Construction)

### CollectorPolicy::create_rem_set()
(#Under Construction)

### PermanentGenerationSpec::init()
See: [here](no78821XK.html) for details






