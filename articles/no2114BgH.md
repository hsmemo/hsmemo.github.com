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
ParallelCompactData::RegionData*
PSParallelCompact::dead_wood_limit_region(const RegionData* beg,
                                          const RegionData* end,
                                          size_t dead_words)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelCompactData& sd = summary_data();
	  size_t left = sd.region(beg);
	  size_t right = end > beg ? sd.region(end) - 1 : left;
	
  {- -------------------------------------------
  (1) 引数の beq と end で指定された範囲中で, 
      その region より前に存在する dead オブジェクトの総量が
      引数で指定された dead_words を超える最初の region を探す.
  
      探索は (left と right という変数を使った) binary search.
      見つかれば, そこでリターン.
    
      (なお, 「その region より前に存在する dead オブジェクトの総量」は, 
       その region 自体のアドレスと
       その region に対応する RegionData の RegionData::destination() との差分を計算すれば分かる.
       以下の dead_to_left の計算を参照)
      ---------------------------------------- -}

	  // Binary search.
	  while (left < right) {
	    // Equivalent to (left + right) / 2, but does not overflow.
	    const size_t middle = left + (right - left) / 2;
	    RegionData* const middle_ptr = sd.region(middle);
	    HeapWord* const dest = middle_ptr->destination();
	    HeapWord* const addr = sd.region_to_addr(middle);
	    assert(dest != NULL, "sanity");
	    assert(dest <= addr, "must move left");
	
	    const size_t dead_to_left = pointer_delta(addr, dest);
	    if (middle > left && dead_to_left > dead_words) {
	      right = middle - 1;
	    } else if (middle < right && dead_to_left < dead_words) {
	      left = middle + 1;
	    } else {
	      return middle_ptr;
	    }
	  }
	  return sd.region(left);
	}
	
```


