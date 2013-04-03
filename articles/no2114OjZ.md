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
void
PSParallelCompact::summarize_space(SpaceId id, bool maximum_compaction)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(id < last_space_id, "id out of range");
	  assert(_space_info[id].dense_prefix() == _space_info[id].space()->bottom() ||
	         ParallelOldGCSplitALot && id == old_space_id,
	         "should have been reset in summarize_spaces_quick()");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const MutableSpace* space = _space_info[id].space();

  {- -------------------------------------------
  (1) 以下の if ブロック内の処理は, 
      もし処理対象の領域内に live オブジェクトが一つも無い場合には (= new_top() と bottom() が等しければ), 省略する
      (PSParallelCompact::summarize_spaces_quick() によって
       SpaceInfo::new_top() にはその領域の live オブジェクト量が格納されていると想定)
      ---------------------------------------- -}

	  if (_space_info[id].new_top() != space->bottom()) {

  {- -------------------------------------------
  (1) PSParallelCompact::compute_dense_prefix() を呼んで, 
      最適な dense prefix のアドレスを取得する (以下の dense_prefix_end).
      (ついでにこのアドレスは, SpaceInfo::set_dense_prefix() で, 対応する SpaceInfo オブジェクト内に記録しておく)
      ---------------------------------------- -}

	    HeapWord* dense_prefix_end = compute_dense_prefix(id, maximum_compaction);
	    _space_info[id].set_dense_prefix(dense_prefix_end);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifndef PRODUCT
	    if (TraceParallelOldGCDensePrefix) {
	      print_dense_prefix_stats("ratio", id, maximum_compaction,
	                               dense_prefix_end);
	      HeapWord* addr = compute_dense_prefix_via_density(id, maximum_compaction);
	      print_dense_prefix_stats("density", id, maximum_compaction, addr);
	    }
	#endif  // #ifndef PRODUCT
	
  {- -------------------------------------------
  (1) 最適な dense prefix を考慮に入れて, 再度 summary data を計算し直す.
      (ただし, 引数の maximum_compaction が true の場合 (= 空き容量が逼迫している場合) は, 
       dense prefix を用意している場合ではないので, この再計算処理は行わない.
       また, dense prefix 量が 0 の場合 (= dense_prefix_end が bottom() と同じ場合) には, 
       再計算しても同じ結果になるので, この再計算処理は行わない.)
  
      再計算処理は, 具体的には以下の処理を行う.
      * PSParallelCompact::fill_dense_prefix_end() を呼んで, 
        dense prefix の終端部分にダミーの live オブジェクトを埋めておく.
        (これは以降の copy/update 処理を簡単にするための処理.
         See: ... #TODO)
      * dense_prefix_end より前の範囲 (= bottom 〜 dense_prefix_end) については, 
        ParallelCompactData::summarize_dense_prefix() で処理する.
        (具体的には, 対応する ParallelCompactData::RegionData の値を変更する)
      * dense_prefix_end より後の範囲 (= dense_prefix_end 〜 top) については, 
        ParallelCompactData::summarize() で処理する.
        (具体的には, 対応する ParallelCompactData::RegionData の値を変更する)
      ---------------------------------------- -}

	    // Recompute the summary data, taking into account the dense prefix.  If
	    // every last byte will be reclaimed, then the existing summary data which
	    // compacts everything can be left in place.
	    if (!maximum_compaction && dense_prefix_end != space->bottom()) {
	      // If dead space crosses the dense prefix boundary, it is (at least
	      // partially) filled with a dummy object, marked live and added to the
	      // summary data.  This simplifies the copy/update phase and must be done
	      // before the final locations of objects are determined, to prevent
	      // leaving a fragment of dead space that is too small to fill.
	      fill_dense_prefix_end(id);
	
	      // Compute the destination of each Region, and thus each object.
	      _summary_data.summarize_dense_prefix(space->bottom(), dense_prefix_end);
	      _summary_data.summarize(_space_info[id].split_info(),
	                              dense_prefix_end, space->top(), NULL,
	                              dense_prefix_end, space->end(),
	                              _space_info[id].new_top_addr());
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCSummaryPhase) {
	    const size_t region_size = ParallelCompactData::RegionSize;
	    HeapWord* const dense_prefix_end = _space_info[id].dense_prefix();
	    const size_t dp_region = _summary_data.addr_to_region_idx(dense_prefix_end);
	    const size_t dp_words = pointer_delta(dense_prefix_end, space->bottom());
	    HeapWord* const new_top = _space_info[id].new_top();
	    const HeapWord* nt_aligned_up = _summary_data.region_align_up(new_top);
	    const size_t cr_words = pointer_delta(nt_aligned_up, dense_prefix_end);
	    tty->print_cr("id=%d cap=" SIZE_FORMAT " dp=" PTR_FORMAT " "
	                  "dp_region=" SIZE_FORMAT " " "dp_count=" SIZE_FORMAT " "
	                  "cr_count=" SIZE_FORMAT " " "nt=" PTR_FORMAT,
	                  id, space->capacity_in_words(), dense_prefix_end,
	                  dp_region, dp_words / region_size,
	                  cr_words / region_size, new_top);
	  }
	}
	
```


