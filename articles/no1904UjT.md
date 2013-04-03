---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/virtualspace.cpp
### 説明(description)

```
// Reserve space for code segment.  Same as Java heap only we mark this as
// executable.
```

### 名前(function name)
```
ReservedCodeSpace::ReservedCodeSpace(size_t r_size,
                                     size_t rs_align,
                                     bool large) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ReservedSpace::ReservedSpace() を呼び出すだけ.
      ---------------------------------------- -}

	  ReservedSpace(r_size, rs_align, large, /*executable*/ true) {
	}
	
```


