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
PSParallelCompact::clear_data_covering_space(SpaceId id)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この段階では, それぞれの Space オブジェクトの top は 
       まだ更新されていないので GC 前の値のままになっている.
       GC 結果を反映した top 位置は, 対応する SpaceInfo オブジェクトの SpaceInfo::new_top() に入っており, 
       GC 処理の最後で Space オブジェクトの top がこの値に修正される.
       
       marking bitmap については, top より上の箇所は(今回の GC 中に使用していないので)クリアする必要は無い.
       summary data についても, top と new_top の大きい方までクリアすれば十分.)
      ---------------------------------------- -}

	  // At this point, top is the value before GC, new_top() is the value that will
	  // be set at the end of GC.  The marking bitmap is cleared to top; nothing
	  // should be marked above top.  The summary data is cleared to the larger of
	  // top & new_top.

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  MutableSpace* const space = _space_info[id].space();
	  HeapWord* const bot = space->bottom();
	  HeapWord* const top = space->top();
	  HeapWord* const max_top = MAX2(top, _space_info[id].new_top());
	
  {- -------------------------------------------
  (1) PSParallelCompact::_mark_bitmap については, 
      ParMarkBitMap::clear_range() で
      処理対象の Space の bottom~top に該当する部分をクリアする.
      ---------------------------------------- -}

	  const idx_t beg_bit = _mark_bitmap.addr_to_bit(bot);
	  const idx_t end_bit = BitMap::word_align_up(_mark_bitmap.addr_to_bit(top));
	  _mark_bitmap.clear_range(beg_bit, end_bit);
	
  {- -------------------------------------------
  (1) PSParallelCompact::_summary_data については, 
      ParallelCompactData::clear_range() で
      処理対象の Space の bottom~top または bottom~new_top の大きい方に該当する部分をクリアする.
      ---------------------------------------- -}

	  const size_t beg_region = _summary_data.addr_to_region_idx(bot);
	  const size_t end_region =
	    _summary_data.addr_to_region_idx(_summary_data.region_align_up(max_top));
	  _summary_data.clear_range(beg_region, end_region);
	
  {- -------------------------------------------
  (1) 処理対象の Space に対応する SplitInfo オブジェクトが GC 中に使用されていれば, 
      (= SplitInfo::is_valid() が true ならば), 
      SplitInfo::clear() で中身をクリアしておく, 
      ---------------------------------------- -}

	  // Clear the data used to 'split' regions.
	  SplitInfo& split_info = _space_info[id].split_info();
	  if (split_info.is_valid()) {
	    split_info.clear();
	  }

  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  DEBUG_ONLY(split_info.verify_clear();)
	}
	
```


