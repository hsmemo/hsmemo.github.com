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
void SplitInfo::record(size_t src_region_idx, size_t partial_obj_size,
                       HeapWord* destination)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(src_region_idx != 0, "invalid src_region_idx");
	  assert(partial_obj_size != 0, "invalid partial_obj_size argument");
	  assert(destination != NULL, "invalid destination argument");
	
  {- -------------------------------------------
  (1) 引数で与えられた情報を, フィールド内に記録しておく.
      ---------------------------------------- -}

	  _src_region_idx = src_region_idx;
	  _partial_obj_size = partial_obj_size;
	  _destination = destination;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // These fields may not be updated below, so make sure they're clear.
	  assert(_dest_region_addr == NULL, "should have been cleared");
	  assert(_first_src_addr == NULL, "should have been cleared");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (beg_region_addr は, (region 境界にアラインメントされたアドレスの中で) 
       コンパクション先での partial object の先頭のアドレスを超えない最大のもの.
       end_region_addr は, (region 境界にアラインメントされたアドレスの中で)
       コンパクション先での partial object の終端のアドレスを超えない最大のもの.)
      ---------------------------------------- -}

	  // Determine the number of destination regions for the partial object.
	  HeapWord* const last_word = destination + partial_obj_size - 1;
	  const ParallelCompactData& sd = PSParallelCompact::summary_data();
	  HeapWord* const beg_region_addr = sd.region_align_down(destination);
	  HeapWord* const end_region_addr = sd.region_align_down(last_word);
	
  {- -------------------------------------------
  (1) 以下のフィールドの値を設定する.
      * _destination_count
      * _dest_region_addr
      * _first_src_addr
    
      設定する値は以下の通り. ただし, 条件については以下のように省略する.
      A -- 「コンパクション先で partial object が 1つの region 内に収まる (つまり beg_region_addr == end_region_addr)」
      B -- 「コンパクション先での partial object の先頭アドレスが region 境界にある (つまり end_region_addr == destination)」
      * _destination_count
        もし, A が成り立つなら 1, 
        そうでなければ 2.
      * _dest_region_addr
        もし, A かつ B なら, end_region_addr, 
        not A の場合も, end_region_addr, 
        それ以外なら, 値の変更は行わない.
      * _first_src_addr
        もし, A かつ B なら, 引数の src_region_idx に対応する region, 
        not A の場合は, 引数の src_region_idx に対応する region に 
          (end_region_addr - destination) 分のオフセットを加えたアドレス, 
        それ以外なら, 値の変更は行わない.
      ---------------------------------------- -}

	  if (beg_region_addr == end_region_addr) {
	    // One destination region.
	    _destination_count = 1;
	    if (end_region_addr == destination) {
	      // The destination falls on a region boundary, thus the first word of the
	      // partial object will be the first word copied to the destination region.
	      _dest_region_addr = end_region_addr;
	      _first_src_addr = sd.region_to_addr(src_region_idx);
	    }
	  } else {
	    // Two destination regions.  When copied, the partial object will cross a
	    // destination region boundary, so a word somewhere within the partial
	    // object will be the first word copied to the second destination region.
	    _destination_count = 2;
	    _dest_region_addr = end_region_addr;
	    const size_t ofs = pointer_delta(end_region_addr, destination);
	    assert(ofs < _partial_obj_size, "sanity");
	    _first_src_addr = sd.region_to_addr(src_region_idx) + ofs;
	  }
	}
	
```


