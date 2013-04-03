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
bool ParallelCompactData::initialize(MemRegion covered_region)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  _region_start = covered_region.start();
	  const size_t region_size = covered_region.word_size();
	  DEBUG_ONLY(_region_end = _region_start + region_size;)
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(region_align_down(_region_start) == _region_start,
	         "region start not aligned");
	  assert((region_size & RegionSizeOffsetMask) == 0,
	         "region size not a multiple of RegionSize");
	
  {- -------------------------------------------
  (1) ParallelCompactData::initialize_region_data() を呼んで, 
      ParallelCompactData::
      ---------------------------------------- -}

	  bool result = initialize_region_data(region_size);
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return result;
	}
	
```


