---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psParallelCompact.cpp
### 説明(description)

```
// Updates the references in the object to their new values.
```

### 名前(function name)
```
ParMarkBitMapClosure::IterationStatus
UpdateOnlyClosure::do_addr(HeapWord* addr, size_t words) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) UpdateOnlyClosure::do_addr() を呼び出すだけ.
      ---------------------------------------- -}

	  do_addr(addr);
	  return ParMarkBitMap::incomplete;
	}
	
```


