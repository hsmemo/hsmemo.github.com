---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
jint G1CollectedHeap::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CollectedHeap::pre_initialize() を呼んで, 初期化の前準備を行っておく.
  
      (現状では C2 用のフィールドの初期化があるだけ)
      ---------------------------------------- -}

	  CollectedHeap::pre_initialize();

  {- -------------------------------------------
  (1) os::enable_vtime() を呼んで, 
      スレッドの "virtual time" (そのスレッドが実際に稼働していた時間) の計測機能を有効にしておく.
    
      (といっても, 現状でこの機能が働くのは Solaris だけ. 他のプラットフォームでは何も起こらない)
      ---------------------------------------- -}

	  os::enable_vtime();
	
  {- -------------------------------------------
  (1) 以降の処理は Heap_lock で排他した状態で行う.
      ---------------------------------------- -}

	  // Necessary to satisfy locking discipline assertions.
	
	  MutexLocker x(Heap_lock);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // While there are no constraints in the GC code that HeapWordSize
	  // be any particular value, there are multiple other areas in the
	  // system which believe this to be true (e.g. oop->object_size in some
	  // cases incorrectly returns the size in wordSize units rather than
	  // HeapWordSize).
	  guarantee(HeapWordSize == wordSize, "HeapWordSize must equal wordSize");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (それぞれヒープ領域の初期サイズ及び最大サイズに相当)
  
      (なお, おかしな値だった場合は Universe::check_alignment() 内で異常終了する)
      ---------------------------------------- -}

	  size_t init_byte_size = collector_policy()->initial_heap_byte_size();
	  size_t max_byte_size = collector_policy()->max_heap_byte_size();
	
	  // Ensure that the sizes are properly aligned.
	  Universe::check_alignment(init_byte_size, HeapRegion::GrainBytes, "g1 heap");
	  Universe::check_alignment(max_byte_size, HeapRegion::GrainBytes, "g1 heap");
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _cg1r = new ConcurrentG1Refine();
	
  {- -------------------------------------------
  (1) ヒープ領域として確保するサイズ, および確保場所として望ましい仮想アドレスを計算する.
      ---------------------------------------- -}

	  // Reserve the maximum.
	  PermanentGenerationSpec* pgs = collector_policy()->permanent_generation();
	  // Includes the perm-gen.
	
	  const size_t total_reserved = max_byte_size + pgs->max_size();
	  char* addr = Universe::preferred_heap_base(total_reserved, Universe::UnscaledNarrowOop);
	
  {- -------------------------------------------
  (1) ヒープ領域をメモリ空間上に確保する
      (なお, ここではまず Young, Old, Perm の全世代分をまとめて1つの連続領域として確保する.
       各 Generation 毎に分ける作業は後で行う.)
      ---------------------------------------- -}

	  ReservedSpace heap_rs(max_byte_size + pgs->max_size(),
	                        HeapRegion::GrainBytes,
	                        UseLargePages, addr);
	
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
	      ReservedSpace heap_rs0(total_reserved, HeapRegion::GrainBytes,
	                             UseLargePages, addr);
	      if (addr != NULL && !heap_rs0.is_reserved()) {
	        // Failed to reserve at specified address again - give up.
	        addr = Universe::preferred_heap_base(total_reserved, Universe::HeapBasedNarrowOop);
	        assert(addr == NULL, "");
	        ReservedSpace heap_rs1(total_reserved, HeapRegion::GrainBytes,
	                               UseLargePages, addr);
	        heap_rs = heap_rs1;
	      } else {
	        heap_rs = heap_rs0;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) もしヒープ領域の確保に失敗していれば, ここでリターン.
      ---------------------------------------- -}

	  if (!heap_rs.is_reserved()) {
	    vm_exit_during_initialization("Could not reserve enough space for object heap");
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) フィールドの初期化
    
      (なおコメントによると, 
      並列に動いているスレッドが (ヒープに何かが入っていると) 誤解しないように初期化するように, 
      とのこと
      ---------------------------------------- -}

	  // It is important to do this in a way such that concurrent readers can't
	  // temporarily think somethings in the heap.  (I've actually seen this
	  // happen in asserts: DLD.)
	  _reserved.set_word_size(0);
	  _reserved.set_start((HeapWord*)heap_rs.base());
	  _reserved.set_end((HeapWord*)(heap_rs.base() + heap_rs.size()));
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _expansion_regions = max_byte_size/HeapRegion::GrainBytes;
	
  {- -------------------------------------------
  (1) 確保したヒープ領域に対応する Remembered Set (CardTableRS オブジェクト) 及び 
      Barrier Set (G1SATBCardTableLoggingModRefBS オブジェクト) を生成する. (See: CardTableRS)
      (そして, それぞれ自分の SharedHeap::_rem_set フィールド, 及び CollectedHeap::_barrier_set フィールドに格納)
      
      もし Barrier Set の確保に失敗したら, ここでリターン.
      ---------------------------------------- -}

	  // Create the gen rem set (and barrier set) for the entire reserved region.
	  _rem_set = collector_policy()->create_rem_set(_reserved, 2);
	  set_barrier_set(rem_set()->bs());
	  if (barrier_set()->is_a(BarrierSet::ModRef)) {
	    _mr_bs = (ModRefBarrierSet*)_barrier_set;
	  } else {
	    vm_exit_during_initialization("G1 requires a mod ref bs.");
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) 確保したヒープ領域に対応する Remembered Set (G1RemSet オブジェクト) を生成する. (See: G1RemSet)
      (そして, 自分の _g1_rem_set フィールドに格納)
      
      もし Remembered Set の確保に失敗したら, ここでリターン.
      ---------------------------------------- -}

	  // Also create a G1 rem set.
	  if (mr_bs()->is_a(BarrierSet::CardTableModRef)) {
	    _g1_rem_set = new G1RemSet(this, (CardTableModRefBS*)mr_bs());
	  } else {
	    vm_exit_during_initialization("G1 requires a cardtable mod ref bs.");
	    return JNI_ENOMEM;
	  }
	
  {- -------------------------------------------
  (1) 上で1つの連続領域として確保したヒープ空間を Perm とそれ以外に分ける.
      (Perm gen に対応する ReservedSpace, および Perm gen 以外に対応する ReservedSpace を作成)
    
      (さらに, Perm Gen 以外に対応する ReservedSpace の方は MemRegion 化して _g1_reserved フィールドに格納しておく)
      ---------------------------------------- -}

	  // Carve out the G1 part of the heap.
	
	  ReservedSpace g1_rs   = heap_rs.first_part(max_byte_size);
	  _g1_reserved = MemRegion((HeapWord*)g1_rs.base(),
	                           g1_rs.size()/HeapWordSize);
	  ReservedSpace perm_gen_rs = heap_rs.last_part(max_byte_size);
	
  {- -------------------------------------------
  (1) PermanentGenerationSpec::init() を呼んで CompactingPermGen を生成し, 
      _perm_gen フィールドにセットする.
      ---------------------------------------- -}

	  _perm_gen = pgs->init(perm_gen_rs, pgs->init_size(), rem_set());
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _g1_storage.initialize(g1_rs, 0);
	  _g1_committed = MemRegion((HeapWord*)_g1_storage.low(), (size_t) 0);
	  _g1_max_committed = _g1_committed;
	  _hrs = new HeapRegionSeq(_expansion_regions);
	  guarantee(_hrs != NULL, "Couldn't allocate HeapRegionSeq");
	
	  // 6843694 - ensure that the maximum region index can fit
	  // in the remembered set structures.
	  const size_t max_region_idx = ((size_t)1 << (sizeof(RegionIdx_t)*BitsPerByte-1)) - 1;
	  guarantee((max_regions() - 1) <= max_region_idx, "too many regions");
	
	  size_t max_cards_per_region = ((size_t)1 << (sizeof(CardIdx_t)*BitsPerByte-1)) - 1;
	  guarantee(HeapRegion::CardsPerRegion > 0, "make sure it's initialized");
	  guarantee((size_t) HeapRegion::CardsPerRegion < max_cards_per_region,
	            "too many cards per region");
	
	  HeapRegionSet::set_unrealistically_long_length(max_regions() + 1);
	
	  _bot_shared = new G1BlockOffsetSharedArray(_reserved,
	                                             heap_word_size(init_byte_size));
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _g1h = this;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	   _in_cset_fast_test_length = max_regions();
	   _in_cset_fast_test_base = NEW_C_HEAP_ARRAY(bool, _in_cset_fast_test_length);
	
	   // We're biasing _in_cset_fast_test to avoid subtracting the
	   // beginning of the heap every time we want to index; basically
	   // it's the same with what we do with the card table.
	   _in_cset_fast_test = _in_cset_fast_test_base -
	                ((size_t) _g1_reserved.start() >> HeapRegion::LogOfHRGrainBytes);
	
	   // Clear the _cset_fast_test bitmap in anticipation of adding
	   // regions to the incremental collection set for the first
	   // evacuation pause.
	   clear_cset_fast_test();
	
	  // Create the ConcurrentMark data structure and thread.
	  // (Must do this late, so that "max_regions" is defined.)
	  _cm       = new ConcurrentMark(heap_rs, (int) max_regions());
	  _cmThread = _cm->cmThread();
	
	  // Initialize the from_card cache structure of HeapRegionRemSet.
	  HeapRegionRemSet::init_heap(max_regions());
	
	  // Now expand into the initial heap size.
	  if (!expand(init_byte_size)) {
	    vm_exit_during_initialization("Failed to allocate initial heap.");
	    return JNI_ENOMEM;
	  }
	
	  // Perform any initialization actions delegated to the policy.
	  g1_policy()->init();
	
	  g1_policy()->note_start_of_mark_thread();
	
	  _refine_cte_cl =
	    new RefineCardTableEntryClosure(ConcurrentG1RefineThread::sts(),
	                                    g1_rem_set(),
	                                    concurrent_g1_refine());
	  JavaThread::dirty_card_queue_set().set_closure(_refine_cte_cl);
	
	  JavaThread::satb_mark_queue_set().initialize(SATB_Q_CBL_mon,
	                                               SATB_Q_FL_lock,
	                                               G1SATBProcessCompletedThreshold,
	                                               Shared_SATB_Q_lock);
	
	  JavaThread::dirty_card_queue_set().initialize(DirtyCardQ_CBL_mon,
	                                                DirtyCardQ_FL_lock,
	                                                concurrent_g1_refine()->yellow_zone(),
	                                                concurrent_g1_refine()->red_zone(),
	                                                Shared_DirtyCardQ_lock);
	
	  if (G1DeferredRSUpdate) {
	    dirty_card_queue_set().initialize(DirtyCardQ_CBL_mon,
	                                      DirtyCardQ_FL_lock,
	                                      -1, // never trigger processing
	                                      -1, // no limit on length
	                                      Shared_DirtyCardQ_lock,
	                                      &JavaThread::dirty_card_queue_set());
	  }
	
	  // Initialize the card queue set used to hold cards containing
	  // references into the collection set.
	  _into_cset_dirty_card_queue_set.initialize(DirtyCardQ_CBL_mon,
	                                             DirtyCardQ_FL_lock,
	                                             -1, // never trigger processing
	                                             -1, // no limit on length
	                                             Shared_DirtyCardQ_lock,
	                                             &JavaThread::dirty_card_queue_set());
	
	  // In case we're keeping closure specialization stats, initialize those
	  // counts and that mechanism.
	  SpecializationStats::clear();
	
	  _gc_alloc_region_list = NULL;
	
	  // Do later initialization work for concurrent refinement.
	  _cg1r->init();
	
	  // Here we allocate the dummy full region that is required by the
	  // G1AllocRegion class. If we don't pass an address in the reserved
	  // space here, lots of asserts fire.
	  MemRegion mr(_g1_reserved.start(), HeapRegion::GrainWords);
	  HeapRegion* dummy_region = new HeapRegion(_bot_shared, mr, true);
	  // We'll re-use the same region whether the alloc region will
	  // require BOT updates or not and, if it doesn't, then a non-young
	  // region will complain that it cannot support allocations without
	  // BOT updates. So we'll tag the dummy region as young to avoid that.
	  dummy_region->set_young();
	  // Make sure it's full.
	  dummy_region->set_top(dummy_region->end());
	  G1AllocRegion::setup(this, dummy_region);
	
	  init_mutator_alloc_region();
	
	  // Do create of the monitoring and management support so that
	  // values in the heap have been properly initialized.
	  _g1mm = new G1MonitoringSupport(this, &_g1_storage);
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JNI_OK;
	}
	
```


