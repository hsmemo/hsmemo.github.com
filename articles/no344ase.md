---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/adjoiningVirtualSpaces.cpp

### 名前(function name)
```
AdjoiningVirtualSpaces::AdjoiningVirtualSpaces(ReservedSpace rs,
                                               size_t min_low_byte_size,
                                               size_t min_high_byte_size,
                                               size_t alignment) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 各フィールドに引数を初期値として設定するだけ
      ---------------------------------------- -}

	  _reserved_space(rs), _min_low_byte_size(min_low_byte_size),
	  _min_high_byte_size(min_high_byte_size), _low(0), _high(0),
	  _alignment(alignment) {}
	
```


