---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp
### 説明(description)

```
// MT-unsafe claiming of a region.  Should only be used during single threaded
// execution.
```

### 名前(function name)
```
inline bool ParallelCompactData::RegionData::claim_unsafe()
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ParallelCompactData::RegionData::available() を呼んで, その結果をリターンする.
      (なお, ParallelCompactData::RegionData::available() が true を返した場合には, 
       _dc_and_los フィールドの dc 部分(destination count部分)の値を dc_claimed に変えておく)
      ---------------------------------------- -}

	  if (available()) {
	    _dc_and_los |= dc_claimed;
	    return true;
	  }
	  return false;
	}
	
```


