---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp

### 名前(function name)
```
void PSParallelCompact::summary_phase(ParCompactionManager* cm,
                                      bool maximum_compaction)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EventMark m("2 summarize");
	  TraceTime tm("summary phase", print_phases(), true, gclog_or_tty);
	  // trace("2");
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifdef  ASSERT
	  if (TraceParallelOldGCMarkingPhase) {
	    tty->print_cr("add_obj_count=" SIZE_FORMAT " "
	                  "add_obj_bytes=" SIZE_FORMAT,
	                  add_obj_count, add_obj_size * HeapWordSize);
	    tty->print_cr("mark_bitmap_count=" SIZE_FORMAT " "
	                  "mark_bitmap_bytes=" SIZE_FORMAT,
	                  mark_bitmap_count, mark_bitmap_size * HeapWordSize);
	  }
	#endif  // #ifdef ASSERT
	
  {- -------------------------------------------
  (1) PSParallelCompact::summarize_spaces_quick() を呼んで, 
      各ヒープ領域中の live オブジェクト量を計算する.
      (この値は, 各ヒープ領域に対応する SpaceInfo オブジェクトの SpaceInfo::new_top() に格納され, 
       以下の処理中で _space_info[*].new_top() で参照されている)
      ---------------------------------------- -}

	  // Quick summarization of each space into itself, to see how much is live.
	  summarize_spaces_quick();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCSummaryPhase) {
	    tty->print_cr("summary_phase:  after summarizing each space to self");
	    Universe::print();
	    NOT_PRODUCT(print_region_ranges());
	    if (Verbose) {
	      NOT_PRODUCT(print_initial_summary_data(_summary_data, _space_info));
	    }
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (old_space_total_live は, コンパクション後に Old 領域に入るオブジェクト量.
       maximum_compaction は, 上記コンパクション後のオブジェクト量(old_space_total_live)が 
       Old 領域の大きさ(capacity)を上回るかどうかを示す boolean(上回るなら true).)
      ---------------------------------------- -}

	  // The amount of live data that will end up in old space (assuming it fits).
	  size_t old_space_total_live = 0;
	  assert(perm_space_id < old_space_id, "should not count perm data here");
	  for (unsigned int id = old_space_id; id < last_space_id; ++id) {
	    old_space_total_live += pointer_delta(_space_info[id].new_top(),
	                                          _space_info[id].space()->bottom());
	  }
	
	  MutableSpace* const old_space = _space_info[old_space_id].space();
	  const size_t old_capacity = old_space->capacity_in_words();
	  if (old_space_total_live > old_capacity) {
	    // XXX - should also try to expand
	    maximum_compaction = true;
	  }

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (ParallelOldGCSplitALot && old_space_total_live < old_capacity) {
	    provoke_split(maximum_compaction);
	  }
	#endif // #ifndef PRODUCT
	
  {- -------------------------------------------
  (1) PSParallelCompact::summarize_space() を2回呼び出して, 
      それぞれ Perm 領域と Old 領域に関して, 
      summary 情報を計算する.
      ---------------------------------------- -}

	  // Permanent and Old generations.
	  summarize_space(perm_space_id, maximum_compaction);
	  summarize_space(old_space_id, maximum_compaction);
	
  {- -------------------------------------------
  (1) 以下の for ループ内で, New 領域内の領域 (Eden, From, To) に対して
      ParallelCompactData::summarize() を呼び出し, 
      それぞれの summary 情報を計算する.
    
      これらの領域については, コンパクション先は Old 領域とする.
      ただし, 全ての live オブジェクトが Old 領域に入りきらない場合は, 
      Old 領域及び自分自身をコンパクション先とする.
    
      (なお, そもそも領域中に live オブジェクトが 1つもない場合(以下の live > 0 が false になる場合)は, 
       その領域については何も処理は行われない)
      ---------------------------------------- -}

	  // Summarize the remaining spaces in the young gen.  The initial target space
	  // is the old gen.  If a space does not fit entirely into the target, then the
	  // remainder is compacted into the space itself and that space becomes the new
	  // target.
	  SpaceId dst_space_id = old_space_id;
	  HeapWord* dst_space_end = old_space->end();
	  HeapWord** new_top_addr = _space_info[dst_space_id].new_top_addr();
	  for (unsigned int id = eden_space_id; id < last_space_id; ++id) {
	    const MutableSpace* space = _space_info[id].space();
	    const size_t live = pointer_delta(_space_info[id].new_top(),
	                                      space->bottom());
	    const size_t available = pointer_delta(dst_space_end, *new_top_addr);
	
	    NOT_PRODUCT(summary_phase_msg(dst_space_id, *new_top_addr, dst_space_end,
	                                  SpaceId(id), space->bottom(), space->top());)

    {- -------------------------------------------
  (1.1) (以下が, 全ての live オブジェクトが Old 領域に入りきる場合の処理)
        (この場合, Old 領域をターゲットとして ParallelCompactData::summarize() が呼び出される)
        ---------------------------------------- -}

	    if (live > 0 && live <= available) {
	      // All the live data will fit.
	      bool done = _summary_data.summarize(_space_info[id].split_info(),
	                                          space->bottom(), space->top(),
	                                          NULL,
	                                          *new_top_addr, dst_space_end,
	                                          new_top_addr);
	      assert(done, "space must fit into old gen");
	
	      // Reset the new_top value for the space.
	      _space_info[id].set_new_top(space->bottom());

    {- -------------------------------------------
  (1.1) (以下が, Old 領域に入りきらない場合の処理)
        (この場合, Old 領域用と自分自身用に, ParallelCompactData::summarize() が2回呼び出される)
        ---------------------------------------- -}

	    } else if (live > 0) {
	      // Attempt to fit part of the source space into the target space.
	      HeapWord* next_src_addr = NULL;
	      bool done = _summary_data.summarize(_space_info[id].split_info(),
	                                          space->bottom(), space->top(),
	                                          &next_src_addr,
	                                          *new_top_addr, dst_space_end,
	                                          new_top_addr);
	      assert(!done, "space should not fit into old gen");
	      assert(next_src_addr != NULL, "sanity");
	
	      // The source space becomes the new target, so the remainder is compacted
	      // within the space itself.
	      dst_space_id = SpaceId(id);
	      dst_space_end = space->end();
	      new_top_addr = _space_info[id].new_top_addr();
	      NOT_PRODUCT(summary_phase_msg(dst_space_id,
	                                    space->bottom(), dst_space_end,
	                                    SpaceId(id), next_src_addr, space->top());)
	      done = _summary_data.summarize(_space_info[id].split_info(),
	                                     next_src_addr, space->top(),
	                                     NULL,
	                                     space->bottom(), dst_space_end,
	                                     new_top_addr);
	      assert(done, "space must fit when compacted into itself");
	      assert(*new_top_addr <= space->top(), "usage should not grow");
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCSummaryPhase) {
	    tty->print_cr("summary_phase:  after final summarization");
	    Universe::print();
	    NOT_PRODUCT(print_region_ranges());
	    if (Verbose) {
	      NOT_PRODUCT(print_generic_summary_data(_summary_data, _space_info));
	    }
	  }
	}
	
```


