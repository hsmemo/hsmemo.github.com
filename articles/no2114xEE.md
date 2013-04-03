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
HeapWord* ParallelCompactData::calc_new_pointer(HeapWord* addr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(addr != NULL, "Should detect NULL oop earlier");
	  assert(PSParallelCompact::gc_heap()->is_in(addr), "addr not in heap");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifdef ASSERT
	  if (PSParallelCompact::mark_bitmap()->is_unmarked(addr)) {
	    gclog_or_tty->print_cr("calc_new_pointer:: addr " PTR_FORMAT, addr);
	  }
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(PSParallelCompact::mark_bitmap()->is_marked(addr), "obj not marked");
	
  {- -------------------------------------------
  (1) まず, 処理対象のオブジェクトが所属しているリージョンの先頭について
      コンパクション後のアドレスを取得する (以下の result).
  
      (このために, まず ParallelCompactData::addr_to_region_idx() と 
       ParallelCompactData::region() で対応する RegionData を取得している. 
       そして, そこから RegionData::destination() で移動先アドレスを出す.)
      ---------------------------------------- -}

	  // Region covering the object.
	  size_t region_index = addr_to_region_idx(addr);
	  const RegionData* const region_ptr = region(region_index);
	  HeapWord* const region_addr = region_align_down(addr);
	
	  assert(addr < region_addr + RegionSize, "Region does not cover object");
	  assert(addr_to_region_ptr(region_addr) == region_ptr, "sanity check");
	
	  HeapWord* result = region_ptr->destination();
	
  {- -------------------------------------------
  (1) 「リージョン先頭の移動先とオブジェクトの移動先との間の差分」を計算し, 
      さっき取得したリージョン先頭の移動先に足し込むことで
      オブジェクトの移動先アドレスを求める (以下の result).
    
      差分の求め方は2通り.
      * 対応するリージョン内に live object しかない場合:
        差分は, コンパクション前と変わらない (リージョン先頭からオブジェクトの先頭までの距離)
      * そうではない場合:
        差分は, 以下の2つの値の合計値.
        * (もし存在するなら) リージョンの先頭に存在する partial object の大きさ
          (これは RegionData::partial_obj_size() で取得)
        * リージョンの先頭(もしくはリージョン先頭の partial object の末尾)から, 
          処理対象のオブジェクトまでの間の, 全 live object の合計量
          (これは ParMarkBitMap::live_words_in_range() で取得)
    
      どちらの場合も, 最終的には求めた移動先をリターン.
      ---------------------------------------- -}

	  // If all the data in the region is live, then the new location of the object
	  // can be calculated from the destination of the region plus the offset of the
	  // object in the region.
	  if (region_ptr->data_size() == RegionSize) {
	    result += pointer_delta(addr, region_addr);
	    DEBUG_ONLY(PSParallelCompact::check_new_location(addr, result);)
	    return result;
	  }
	
	  // The new location of the object is
	  //    region destination +
	  //    size of the partial object extending onto the region +
	  //    sizes of the live objects in the Region that are to the left of addr
	  const size_t partial_obj_size = region_ptr->partial_obj_size();
	  HeapWord* const search_start = region_addr + partial_obj_size;
	
	  const ParMarkBitMap* bitmap = PSParallelCompact::mark_bitmap();
	  size_t live_to_left = bitmap->live_words_in_range(search_start, oop(addr));
	
	  result += partial_obj_size + live_to_left;
	  DEBUG_ONLY(PSParallelCompact::check_new_location(addr, result);)
	  return result;
	}
	
```


