---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp

### 名前(function name)
```
jint ParallelScavengeHeap::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedHeap::pre_initialize() を呼んで, 初期化の前準備を行っておく.
  
      (現状では C2 用のフィールドの初期化があるだけ)
      ---------------------------------------- -}

	  CollectedHeap::pre_initialize();
	
  {- -------------------------------------------
  (1) ParallelScavengeHeap 用の CollectorPolicy (GenerationSizer オブジェクト) を作成し, 
      _collector_policy フィールドに格納.
      (See: GenerationSizer)
      ---------------------------------------- -}

	  // Cannot be initialized until after the flags are parsed
	  // GenerationSizer flag_parser;
	  _collector_policy = new GenerationSizer();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (それぞれ Xms, Xmx 等で指定された young, old perm 世代の最小値/最大値に相当)
      ---------------------------------------- -}

	  size_t yg_min_size = _collector_policy->min_young_gen_size();
	  size_t yg_max_size = _collector_policy->max_young_gen_size();
	  size_t og_min_size = _collector_policy->min_old_gen_size();
	  size_t og_max_size = _collector_policy->max_old_gen_size();
	  // Why isn't there a min_perm_gen_size()?
	  size_t pg_min_size = _collector_policy->perm_gen_size();
	  size_t pg_max_size = _collector_policy->max_perm_gen_size();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  trace_gen_sizes("ps heap raw",
	                  pg_min_size, pg_max_size,
	                  og_min_size, og_max_size,
	                  yg_min_size, yg_max_size);
	
  {- -------------------------------------------
  (1) ヒープ領域として確保するサイズを計算する
  
      (以下のような値を計算する (あるいは既に計算していた値を微修正する).
        og_cur_size, og_min_size, og_max_size, og_align, 
        yg_cur_size, yg_min_size, yg_max_size, yg_align, 
        pg_cur_size, pg_min_size, pg_max_size, pg_align)
  
      (なお, ReservedSpace のコンストラクタの都合により, perm gen のサイズは
       その他の世代(young,old)の合計サイズ以下でないといけないらしい)
      ---------------------------------------- -}

	  // The ReservedSpace ctor used below requires that the page size for the perm
	  // gen is <= the page size for the rest of the heap (young + old gens).
	  const size_t og_page_sz = os::page_size_for_region(yg_min_size + og_min_size,
	                                                     yg_max_size + og_max_size,
	                                                     8);
	  const size_t pg_page_sz = MIN2(os::page_size_for_region(pg_min_size,
	                                                          pg_max_size, 16),
	                                 og_page_sz);
	
	  const size_t pg_align = set_alignment(_perm_gen_alignment,  pg_page_sz);
	  const size_t og_align = set_alignment(_old_gen_alignment,   og_page_sz);
	  const size_t yg_align = set_alignment(_young_gen_alignment, og_page_sz);
	
	  // Update sizes to reflect the selected page size(s).
	  //
	  // NEEDS_CLEANUP.  The default TwoGenerationCollectorPolicy uses NewRatio; it
	  // should check UseAdaptiveSizePolicy.  Changes from generationSizer could
	  // move to the common code.
	  yg_min_size = align_size_up(yg_min_size, yg_align);
	  yg_max_size = align_size_up(yg_max_size, yg_align);
	  size_t yg_cur_size =
	    align_size_up(_collector_policy->young_gen_size(), yg_align);
	  yg_cur_size = MAX2(yg_cur_size, yg_min_size);
	
	  og_min_size = align_size_up(og_min_size, og_align);
	  // Align old gen size down to preserve specified heap size.
	  assert(og_align == yg_align, "sanity");
	  og_max_size = align_size_down(og_max_size, og_align);
	  og_max_size = MAX2(og_max_size, og_min_size);
	  size_t og_cur_size =
	    align_size_down(_collector_policy->old_gen_size(), og_align);
	  og_cur_size = MAX2(og_cur_size, og_min_size);
	
	  pg_min_size = align_size_up(pg_min_size, pg_align);
	  pg_max_size = align_size_up(pg_max_size, pg_align);
	  size_t pg_cur_size = pg_min_size;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  trace_gen_sizes("ps heap rnd",
	                  pg_min_size, pg_max_size,
	                  og_min_size, og_max_size,
	                  yg_min_size, yg_max_size);
	
  {- -------------------------------------------
  (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する
      ---------------------------------------- -}

	  const size_t total_reserved = pg_max_size + og_max_size + yg_max_size;
	  char* addr = Universe::preferred_heap_base(total_reserved, Universe::UnscaledNarrowOop);
	
  {- -------------------------------------------
  (1) ヒープ領域をメモリ空間上に確保する
      (なお, ここではまず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保する.
       各 Generation 毎に分ける作業は後で行う.)
      ---------------------------------------- -}

	  // The main part of the heap (old gen + young gen) can often use a larger page
	  // size than is needed or wanted for the perm gen.  Use the "compound
	  // alignment" ReservedSpace ctor to avoid having to use the same page size for
	  // all gens.
	
	  ReservedHeapSpace heap_rs(pg_max_size, pg_align, og_max_size + yg_max_size,
	                            og_align, addr);
	
  {- -------------------------------------------
  (1) もし UseCompressedOops オプションが指定されており, かつ指定した仮想アドレスでの領域確保が失敗していた場合は
      アドレスを変えて再度確保を試みる (Universe::ZeroBasedNarrowOop).
      それでもダメなら, heap base を用いる方法で確保を試みる (Universe::HeapBasedNarrowOop).
      ---------------------------------------- -}

	  if (UseCompressedOops) {
	    if (addr != NULL && !heap_rs.is_reserved()) {
	      // Failed to reserve at specified address - the requested memory
	      // region is taken already, for example, by 'java' launcher.
	      // Try again to reserver heap higher.
	      addr = Universe::preferred_heap_base(total_reserved, Universe::ZeroBasedNarrowOop);
	      ReservedHeapSpace heap_rs0(pg_max_size, pg_align, og_max_size + yg_max_size,
	                                 og_align, addr);
	      if (addr != NULL && !heap_rs0.is_reserved()) {
	        // Failed to reserve at specified address again - give up.
	        addr = Universe::preferred_heap_base(total_reserved, Universe::HeapBasedNarrowOop);
	        assert(addr == NULL, "");
	        ReservedHeapSpace heap_rs1(pg_max_size, pg_align, og_max_size + yg_max_size,
	                                   og_align, addr);
	        heap_rs = heap_rs1;
	      } else {
	        heap_rs = heap_rs0;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  os::trace_page_sizes("ps perm", pg_min_size, pg_max_size, pg_page_sz,
	                       heap_rs.base(), pg_max_size);
	  os::trace_page_sizes("ps main", og_min_size + yg_min_size,
	                       og_max_size + yg_max_size, og_page_sz,
	                       heap_rs.base() + pg_max_size,
	                       heap_rs.size() - pg_max_size);

  {- -------------------------------------------
  (1) もしヒープ領域の確保に失敗していたら, ここでリターン.
      ---------------------------------------- -}

	  if (!heap_rs.is_reserved()) {
	    vm_shutdown_during_initialization(
	      "Could not reserve enough space for object heap");
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) 確保したヒープ領域を表す MemRegion オブジェクトを生成し, 
      自分の CollectedHeap::_reserved フィールドに格納.
      ---------------------------------------- -}

	  _reserved = MemRegion((HeapWord*)heap_rs.base(),
	                        (HeapWord*)(heap_rs.base() + heap_rs.size()));
	
  {- -------------------------------------------
  (1) 確保したヒープ領域に対応する Barrier Set (CardTableExtension オブジェクト) を生成する. (See: CardTableExtension)
      (そして, 自分の CollectedHeap::_barrier_set フィールドに格納)
      (また, oopDesc のフィールドにも格納しているようだが... #TODO)
      
      もし Barrier Set の確保に失敗したら, ここでリターン.
      ---------------------------------------- -}

	  CardTableExtension* const barrier_set = new CardTableExtension(_reserved, 3);
	  _barrier_set = barrier_set;
	  oopDesc::set_bs(_barrier_set);
	  if (_barrier_set == NULL) {
	    vm_shutdown_during_initialization(
	      "Could not reserve enough space for barrier set");
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Initial young gen size is 4 Mb
	  //
	  // XXX - what about flag_parser.young_gen_size()?
	  const size_t init_young_size = align_size_up(4 * M, yg_align);
	  yg_cur_size = MAX2(MIN2(init_young_size, yg_max_size), yg_cur_size);
	
  {- -------------------------------------------
  (1) 上で1つの連続領域として確保したヒープ空間を Perm とそれ以外に分ける.
      (Perm gen に対応する ReservedSpace, および Perm gen 以外に対応する ReservedSpace を作成)
      ---------------------------------------- -}

	  // Split the reserved space into perm gen and the main heap (everything else).
	  // The main heap uses a different alignment.
	  ReservedSpace perm_rs = heap_rs.first_part(pg_max_size);
	  ReservedSpace main_rs = heap_rs.last_part(pg_max_size, og_align);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (GC Ergonomics 用.
       See: GC Ergonomics)
      ---------------------------------------- -}

	  // Make up the generations
	  // Calculate the maximum size that a generation can grow.  This
	  // includes growth into the other generation.  Note that the
	  // parameter _max_gen_size is kept as the maximum
	  // size of the generation as the boundaries currently stand.
	  // _max_gen_size is still used as that value.
	  double max_gc_pause_sec = ((double) MaxGCPauseMillis)/1000.0;
	  double max_gc_minor_pause_sec = ((double) MaxGCMinorPauseMillis)/1000.0;
	
  {- -------------------------------------------
  (1) さらに, Perm gen 以外の領域を Old と Young に分け, 対応する Generation オブジェクトを生成する.
      (PSOldGen オブジェクトと PSYoungGen オブジェクトを生成し, それぞれ _old_gen, _young_gen フィールドに格納)
      ---------------------------------------- -}

	  _gens = new AdjoiningGenerations(main_rs,
	                                   og_cur_size,
	                                   og_min_size,
	                                   og_max_size,
	                                   yg_cur_size,
	                                   yg_min_size,
	                                   yg_max_size,
	                                   yg_align);
	
	  _old_gen = _gens->old_gen();
	  _young_gen = _gens->young_gen();
	
  {- -------------------------------------------
  (1) ParallelScavengeHeap 用の AdaptiveSizePolicy (PSAdaptiveSizePolicy オブジェクト) を作成し, 
      _size_policy フィールドに格納. (See: PSAdaptiveSizePolicy)
      ---------------------------------------- -}

	  const size_t eden_capacity = _young_gen->eden_space()->capacity_in_bytes();
	  const size_t old_capacity = _old_gen->capacity_in_bytes();
	  const size_t initial_promo_size = MIN2(eden_capacity, old_capacity);
	  _size_policy =
	    new PSAdaptiveSizePolicy(eden_capacity,
	                             initial_promo_size,
	                             young_gen()->to_space()->capacity_in_bytes(),
	                             intra_heap_alignment(),
	                             max_gc_pause_sec,
	                             max_gc_minor_pause_sec,
	                             GCTimeRatio
	                             );
	
  {- -------------------------------------------
  (1) Perm についても Generation オブジェクト (PSPermGen) を作成し, _perm_gen フィールドに格納.
      ---------------------------------------- -}

	  _perm_gen = new PSPermGen(perm_rs,
	                            pg_align,
	                            pg_cur_size,
	                            pg_cur_size,
	                            pg_max_size,
	                            "perm", 2);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!UseAdaptiveGCBoundary ||
	    (old_gen()->virtual_space()->high_boundary() ==
	     young_gen()->virtual_space()->low_boundary()),
	    "Boundaries must meet");

  {- -------------------------------------------
  (1) ParallelScavengeHeap 用の GCAdaptivePolicyCounters (PSGCAdaptivePolicyCounters オブジェクト) を作成し, 
      _gc_policy_counters フィールドに格納. (See: PSGCAdaptivePolicyCounters)
      ---------------------------------------- -}

	  // initialize the policy counters - 2 collectors, 3 generations
	  _gc_policy_counters =
	    new PSGCAdaptivePolicyCounters("ParScav:MSC", 2, 3, _size_policy);

  {- -------------------------------------------
  (1) static 変数の _psh に自分自身をセットしておく.
      ---------------------------------------- -}

	  _psh = this;
	
  {- -------------------------------------------
  (1) GCTaskManager::create() で, GC を並列に実行するための作業用スレッドを生成する.
      ---------------------------------------- -}

	  // Set up the GCTaskManager
	  _gc_task_manager = GCTaskManager::create(ParallelGCThreads);
	
  {- -------------------------------------------
  (1) UseParallelOldGC オプションが指定されていれば, PSParallelCompact::initialize() で初期化を行う.
      初期化が失敗したらここでリターン.
      ---------------------------------------- -}

	  if (UseParallelOldGC && !PSParallelCompact::initialize()) {
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JNI_OK;
	}
	
```


