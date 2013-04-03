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
PSParallelCompact::first_dead_space_region(const RegionData* beg,
                                           const RegionData* end)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const size_t region_size = ParallelCompactData::RegionSize;
	  ParallelCompactData& sd = summary_data();
	  size_t left = sd.region(beg);
	  size_t right = end > beg ? sd.region(end) - 1 : left;
	
  {- -------------------------------------------
  (1) 引数の beq と end で指定された範囲で, 最初の dead オブジェクトが存在する region を探す.
      探索は (left と right という変数を使った) binary search.
      見つかれば, そこでリターン.
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
	
	    if (middle > left && dest < addr) {
	      right = middle - 1;
	    } else if (middle < right && middle_ptr->data_size() == region_size) {
	      left = middle + 1;
	    } else {
	      return middle_ptr;
	    }
	  }

  {- -------------------------------------------
  (1) left と right が一致してしまったら, left のアドレスをリターンする.
      (dead オブジェクトがいなかった場合とか, そもそも beg と end が 1つの region 内に収まっていた場合とか)
      ---------------------------------------- -}

	  return sd.region(left);
	}
	
```


