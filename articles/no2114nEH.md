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
bool ParallelCompactData::summarize(SplitInfo& split_info,
                                    HeapWord* source_beg, HeapWord* source_end,
                                    HeapWord** source_next,
                                    HeapWord* target_beg, HeapWord* target_end,
                                    HeapWord** target_next)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceParallelOldGCSummaryPhase) {
	    HeapWord* const source_next_val = source_next == NULL ? NULL : *source_next;
	    tty->print_cr("sb=" PTR_FORMAT " se=" PTR_FORMAT " sn=" PTR_FORMAT
	                  "tb=" PTR_FORMAT " te=" PTR_FORMAT " tn=" PTR_FORMAT,
	                  source_beg, source_end, source_next_val,
	                  target_beg, target_end, *target_next);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (cur_region は, 現在処理している region (ループ中で更新していく)
       end_region は, 処理範囲の終端を示す region.
       dest_addr は, 次に処理するオブジェクトのためのコンパクション先のアドレス (ループ中で更新していく))
      ---------------------------------------- -}

	  size_t cur_region = addr_to_region_idx(source_beg);
	  const size_t end_region = addr_to_region_idx(region_align_up(source_end));
	
	  HeapWord *dest_addr = target_beg;

  {- -------------------------------------------
  (1) 移動元の全 region (引数の source_beg から source_end まで) に対して, 
      以下の while ループ内で処理を行う
  
      このループ内では, 対象範囲に含まれる全ての region について
      対応する ParallelCompactData::RegionData オブジェクトに (あるいは
      処理対象の region がコンパクションされる先の ParallelCompactData::RegionData オブジェクトに)
      以下の関数で情報を設定する.
      * ParallelCompactData::RegionData::set_destination()
      * ParallelCompactData::RegionData::set_destination_count()
      * ParallelCompactData::RegionData::set_source_region()
      * ParallelCompactData::RegionData::set_data_location()
      ---------------------------------------- -}

	  while (cur_region < end_region) {

    {- -------------------------------------------
  (1.1) ParallelCompactData::RegionData::set_destination() で, 
        コンパクション先のアドレスを dest_addr に設定する.
        ---------------------------------------- -}

	    // The destination must be set even if the region has no data.
	    _region_data[cur_region].set_destination(dest_addr);
	
    {- -------------------------------------------
  (1.1) もし, 処理対象の region に live オブジェクトが 1つもなければ, 
        この処理対象の region については特にすることはないので,
        以下の if ブロック内の処理は省略する.
        ---------------------------------------- -}

	    size_t words = _region_data[cur_region].data_size();
	    if (words > 0) {

      {- -------------------------------------------
  (1.1.1) もし, 処理対象の region 内の live オブジェクトが
          コンパクション先の Space には入りきらない場合, 
          これ以上の処理は出来ないので, ここでリターン.
          (なお, リターンする前に
           ParallelCompactData::summarize_split_space() を呼んで
           適当な分割箇所を記録しておく)
          ---------------------------------------- -}

	      // If cur_region does not fit entirely into the target space, find a point
	      // at which the source space can be 'split' so that part is copied to the
	      // target space and the rest is copied elsewhere.
	      if (dest_addr + words > target_end) {
	        assert(source_next != NULL, "source_next is NULL when splitting");
	        *source_next = summarize_split_space(cur_region, split_info, dest_addr,
	                                             target_end, target_next);
	        return false;
	      }
	
      {- -------------------------------------------
  (1.1.1) 以下の関数を呼んで, ParallelCompactData::RegionData オブジェクトに情報を設定する.
          * ParallelCompactData::RegionData::set_destination_count()
          * ParallelCompactData::RegionData::set_source_region()
          * ParallelCompactData::RegionData::set_data_location()
  
  
          設定対象, および設定内容は以下の通り. ただし, 条件については以下のように省略する.
          A -- 「処理対象の region が, (コンパクション先を別領域に切り替える)分割点を含んでいる.
                 (SplitInfo.is_split() が true)」
                (なおこの条件は, コンパクション先の Space に入りきらないために一旦諦めた後に, 
                 再度コンパクション先を変えて ParallelCompactData::summarize() が呼び出し直された場合にのみ成り立つ.
                 See: ParallelCompactData::summarize_split_space(), 
                      SplitInfo::record(), 
                      PSParallelCompact::summary_phase())
          A′ -- A であり, かつコンパクション先が 2つの region にまたがる.
          B -- 「コンパクション先の region (のうちの少なくとも1つ) は, 処理対象の region である.
                 (つまり cur_region == dest_region_2)」
          C -- 「処理対象の region のコンパクション先が, 2つの region にまたがる.
                 (つまり dest_region_1 != dest_region_2)」
          D -- 「コンパクション先の先頭のアドレスが region のちょうど境界部分にある.
                 (つまり, region_offset(dest_addr) == 0)」
  
          * ParallelCompactData::RegionData::set_destination_count()
            処理対象の region (cur_reigon) に対して設定を行う.
  
            設定する値は以下の通り (詳細は, 以下の destination_count 参照).
            * A の場合, ???
            * not A &&     B &&     C の場合, 1
            * not A && not B &&     C の場合, 2
            * not A && not B && not C の場合, 1
            * それ以外の場合, 0
  
            (SplitInfo::is_split() が true の場合の destination_count はどうなる?? 
             2 を超えることもありうる??? #TODO)
  
          * ParallelCompactData::RegionData::set_source_region()
            処理対象の region (cur_reigon) のコンパクション先の region に対して設定を行う.
            設定する値(= コンパクション元)は cur_region.
            
            設定対象となる ParallelCompactData::RegionData オブジェクトは以下のように選択する.
            なお, ParallelCompactData::RegionData::set_source_region() については
            条件を満たしているものが複数あれば, それら全てに対して呼ばれる.
            (逆に, 条件を満たしているものが 1つもなければ 1度も呼ばれない)
            * A′が成り立つ場合, 2つの region のうちの後ろの方 (以下の dest_idx) を対象として呼ばれる.
            * C が成り立つ場合, 2つの region のうちの後ろの方 (以下の dest_region_2) を対象として呼ばれる.
            * D が成り立つ場合, コンパクション先の region (以下の dest_region_1) を対象として呼ばれる.
              (なお, C かつ D はない. コンパクション後の量が 1 region 分を超えることはあり得ないので)
  
          * ParallelCompactData::RegionData::set_data_location()
            処理対象の region (cur_reigon) に対して設定を行う.
    
            設定する値(= 対応するアドレス)は, 
            (どの場合においても) 現在の自分自身のアドレス (以下の region_to_addr(cur_region)) とする.
          ---------------------------------------- -}

	      // Compute the destination_count for cur_region, and if necessary, update
	      // source_region for a destination region.  The source_region field is
	      // updated if cur_region is the first (left-most) region to be copied to a
	      // destination region.
	      //
	      // The destination_count calculation is a bit subtle.  A region that has
	      // data that compacts into itself does not count itself as a destination.
	      // This maintains the invariant that a zero count means the region is
	      // available and can be claimed and then filled.
	      uint destination_count = 0;
	      if (split_info.is_split(cur_region)) {
	        // The current region has been split:  the partial object will be copied
	        // to one destination space and the remaining data will be copied to
	        // another destination space.  Adjust the initial destination_count and,
	        // if necessary, set the source_region field if the partial object will
	        // cross a destination region boundary.
	        destination_count = split_info.destination_count();
	        if (destination_count == 2) {
	          size_t dest_idx = addr_to_region_idx(split_info.dest_region_addr());
	          _region_data[dest_idx].set_source_region(cur_region);
	        }
	      }
	
	      HeapWord* const last_addr = dest_addr + words - 1;
	      const size_t dest_region_1 = addr_to_region_idx(dest_addr);
	      const size_t dest_region_2 = addr_to_region_idx(last_addr);
	
	      // Initially assume that the destination regions will be the same and
	      // adjust the value below if necessary.  Under this assumption, if
	      // cur_region == dest_region_2, then cur_region will be compacted
	      // completely into itself.
	      destination_count += cur_region == dest_region_2 ? 0 : 1;
	      if (dest_region_1 != dest_region_2) {
	        // Destination regions differ; adjust destination_count.
	        destination_count += 1;
	        // Data from cur_region will be copied to the start of dest_region_2.
	        _region_data[dest_region_2].set_source_region(cur_region);
	      } else if (region_offset(dest_addr) == 0) {
	        // Data from cur_region will be copied to the start of the destination
	        // region.
	        _region_data[dest_region_1].set_source_region(cur_region);
	      }
	
	      _region_data[cur_region].set_destination_count(destination_count);
	      _region_data[cur_region].set_data_location(region_to_addr(cur_region));

      {- -------------------------------------------
  (1.1.1) 今回処理したオブジェクトの大きさ分だけ dest_addr を進める.
          ---------------------------------------- -}

	      dest_addr += words;
	    }
	
    {- -------------------------------------------
  (1.1) 処理対象を次の region へ進める.
        ---------------------------------------- -}

	    ++cur_region;
	  }
	
  {- -------------------------------------------
  (1) リターンする.
      (なおリターンの直前に, 引数で渡された target_next に dest_addr を書き込んでおく)
      ---------------------------------------- -}

	  *target_next = dest_addr;
	  return true;
	}
	
```


