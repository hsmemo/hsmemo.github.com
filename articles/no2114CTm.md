---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
### 説明(description)
(この関数では, ある region 内の live オブジェクト全てを
 一つの Space 内にはコンパクション出来ない場合に, 
 コンパクション先を別の Space へと切り替える分割点をどこにするか, を計算する.
 (See: SplitInfo)
 
 もし, 問題となっている region が partial object (もっと前の region から始まって
 現在の region まではみ出ているオブジェクト) を持たなければ, 
 region の先頭を分割点とする.

 partial object がある場合は, 
 partial object が始まった region まで遡って分割点を入れる.
 分割点は以下のように決める.
 * 問題の region が partial object を持たない場合,  
   その region の先頭を分割点とする.
 * partial object がある場合は, partial object 分だけを現在のコンパクション先にコンパクションし, 
   それ以降は別の Space にコンパクションする.
   (つまり, partial object の直後を分割点とする) 

```
// Find the point at which a space can be split and, if necessary, record the
// split point.
//
// If the current src region (which overflowed the destination space) doesn't
// have a partial object, the split point is at the beginning of the current src
// region (an "easy" split, no extra bookkeeping required).
//
// If the current src region has a partial object, the split point is in the
// region where that partial object starts (call it the split_region).  If
// split_region has a partial object, then the split point is just after that
// partial object (a "hard" split where we have to record the split data and
// zero the partial_obj_size field).  With a "hard" split, we know that the
// partial_obj ends within split_region because the partial object that caused
// the overflow starts in split_region.  If split_region doesn't have a partial
// obj, then the split is at the beginning of split_region (another "easy"
// split).
```

### 名前(function name)
```
HeapWord*
ParallelCompactData::summarize_split_space(size_t src_region,
                                           SplitInfo& split_info,
                                           HeapWord* destination,
                                           HeapWord* target_end,
                                           HeapWord** target_next)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(destination <= target_end, "sanity");
	  assert(destination + _region_data[src_region].data_size() > target_end,
	    "region should not fit into target space");
	  assert(is_region_aligned(target_end), "sanity");
	
  {- -------------------------------------------
  (1) まず, 分割点の箇所を計算する.
  
      (以下の split_region 変数は, 分割点を入れる region を示す.
       また, partial_obj_size 変数は, split_region 内の分割点の位置(先頭からのオフセット)を示す.
  
       通常は, 引数で渡された src_region が split_region になり, 分割点は以下のように決まる.
       * src_region が partial object (もっと前の region から始まって
         現在の region まではみ出ているオブジェクト) を持たない場合,  
         src_region の先頭を分割点とする.
       * partial object がある場合は, partial object 分だけを現在のコンパクション先にコンパクションし, 
         それ以降は別の Space にコンパクションする.
         (つまり, partial object の直後を分割点とする)
  
       ただし, コンパクション先に partial object を入れる空き領域さえない場合は, 
       問題の partial object が始まった region まで遡って分割点を入れる.
       この場合, split_region は partial object が始まった region になる.
       そして, 分割点は以下のように決まる.
       * 問題の region が partial object を持たない場合,  
         その region の先頭を分割点とする.
       * partial object がある場合は, partial object 分だけを現在のコンパクション先にコンパクションし, 
         それ以降は別の Space にコンパクションする.
         (つまり, partial object の直後を分割点とする)
       なおこの場合は, 分割点以降の範囲については, 既にコンパクション先が決まっていたのに
       コンパクション先を別の領域へと変更することになるので, 
       対応するコンパクション先の ParallelCompactData::RegionData オブジェクト全てに対して
       ParallelCompactData::RegionData::set_source_region() で 
       コンパクション元情報を 0 にリセットしておく.
       (ついでに(トレース出力)も行う))
      ---------------------------------------- -}

	  size_t split_region = src_region;
	  HeapWord* split_destination = destination;
	  size_t partial_obj_size = _region_data[src_region].partial_obj_size();
	
	  if (destination + partial_obj_size > target_end) {
	    // The split point is just after the partial object (if any) in the
	    // src_region that contains the start of the object that overflowed the
	    // destination space.
	    //
	    // Find the start of the "overflow" object and set split_region to the
	    // region containing it.
	    HeapWord* const overflow_obj = _region_data[src_region].partial_obj_addr();
	    split_region = addr_to_region_idx(overflow_obj);
	
	    // Clear the source_region field of all destination regions whose first word
	    // came from data after the split point (a non-null source_region field
	    // implies a region must be filled).
	    //
	    // An alternative to the simple loop below:  clear during post_compact(),
	    // which uses memcpy instead of individual stores, and is easy to
	    // parallelize.  (The downside is that it clears the entire RegionData
	    // object as opposed to just one field.)
	    //
	    // post_compact() would have to clear the summary data up to the highest
	    // address that was written during the summary phase, which would be
	    //
	    //         max(top, max(new_top, clear_top))
	    //
	    // where clear_top is a new field in SpaceInfo.  Would have to set clear_top
	    // to target_end.
	    const RegionData* const sr = region(split_region);
	    const size_t beg_idx =
	      addr_to_region_idx(region_align_up(sr->destination() +
	                                         sr->partial_obj_size()));
	    const size_t end_idx = addr_to_region_idx(target_end);
	
	    if (TraceParallelOldGCSummaryPhase) {
	        gclog_or_tty->print_cr("split:  clearing source_region field in ["
	                               SIZE_FORMAT ", " SIZE_FORMAT ")",
	                               beg_idx, end_idx);
	    }
	    for (size_t idx = beg_idx; idx < end_idx; ++idx) {
	      _region_data[idx].set_source_region(0);
	    }
	
	    // Set split_destination and partial_obj_size to reflect the split region.
	    split_destination = sr->destination();
	    partial_obj_size = sr->partial_obj_size();
	  }
	
  {- -------------------------------------------
  (1) もし, 分割点を入れる region に partial object がある場合は, 
      ParallelCompactData::RegionData::set_partial_obj_size() で
      partial object 量情報を 0 にリセットする代わりに, 
      SplitInfo::record() で partial object の直後のアドレスを記録しておく.
      ---------------------------------------- -}

	  // The split is recorded only if a partial object extends onto the region.
	  if (partial_obj_size != 0) {
	    _region_data[split_region].set_partial_obj_size(0);
	    split_info.record(split_region, partial_obj_size, split_destination);
	  }
	
  {- -------------------------------------------
  (1) 引数で渡された target_next に, コンパクション先の領域に関する
      コンパクション後の使用量情報(新しい top 位置のアドレス)を記録しておく.
      ---------------------------------------- -}

	  // Setup the continuation addresses.
	  *target_next = split_destination + partial_obj_size;

  {- -------------------------------------------
  (1) (変数宣言など)
      (source_next が, 実際の分割点のアドレス)
      ---------------------------------------- -}

	  HeapWord* const source_next = region_to_addr(split_region) + partial_obj_size;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCSummaryPhase) {
	    const char * split_type = partial_obj_size == 0 ? "easy" : "hard";
	    gclog_or_tty->print_cr("%s split:  src=" PTR_FORMAT " src_c=" SIZE_FORMAT
	                           " pos=" SIZE_FORMAT,
	                           split_type, source_next, split_region,
	                           partial_obj_size);
	    gclog_or_tty->print_cr("%s split:  dst=" PTR_FORMAT " dst_c=" SIZE_FORMAT
	                           " tn=" PTR_FORMAT,
	                           split_type, split_destination,
	                           addr_to_region_idx(split_destination),
	                           *target_next);
	
	    if (partial_obj_size != 0) {
	      HeapWord* const po_beg = split_info.destination();
	      HeapWord* const po_end = po_beg + split_info.partial_obj_size();
	      gclog_or_tty->print_cr("%s split:  "
	                             "po_beg=" PTR_FORMAT " " SIZE_FORMAT " "
	                             "po_end=" PTR_FORMAT " " SIZE_FORMAT,
	                             split_type,
	                             po_beg, addr_to_region_idx(po_beg),
	                             po_end, addr_to_region_idx(po_end));
	    }
	  }
	
  {- -------------------------------------------
  (1) 分割点のアドレス(source_next)をリターン
      ---------------------------------------- -}

	  return source_next;
	}
	
```


