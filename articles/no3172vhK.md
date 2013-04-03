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
bool ParallelCompactData::initialize_region_data(size_t region_size)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const size_t count = (region_size + RegionSizeOffsetMask) >> Log2RegionSize;

  {- -------------------------------------------
  (1) ParallelCompactData::create_vspace() を呼んで, 
      _region_data 用のメモリ領域の確保を試みる.
      ---------------------------------------- -}

	  _region_vspace = create_vspace(count, sizeof(RegionData));

  {- -------------------------------------------
  (1) メモリ領域の確保に成功すれば, そのアドレスを _region_data フィールドにセットし, 
      (ついでに _region_count フィールドの値もセットし, )
      true をリターン.
  
      逆にメモリの確保に失敗した場合は, 何もせずに false をリターンするだけ.
      ---------------------------------------- -}

	  if (_region_vspace != 0) {
	    _region_data = (RegionData*)_region_vspace->reserved_low_addr();
	    _region_count = count;
	    return true;
	  }
	  return false;
	}
	
```


