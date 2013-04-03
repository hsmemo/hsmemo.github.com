---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.hpp

### 名前(function name)
```
inline bool SplitInfo::is_split(size_t region_idx) const
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数の region_idx が _src_region_idx フィールドに記録されている値に一致し, 
      かつ _src_region_idx フィールドの値が有効なものであれば (SplitInfo::is_valid() が true であれば), 
      true を返す.
      (See: SplitInfo::record())
      ---------------------------------------- -}

	  return _src_region_idx == region_idx && is_valid();
	}
	
```


