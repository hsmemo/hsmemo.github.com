---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/parallelScavengeHeap.cpp

### 名前(function name)
```
bool ParallelScavengeHeap::is_in_reserved(const void* p) const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数のポインタが, New 領域か Old 領域か Perm 領域の中にあれば true.
      そうでなければ false.
      ---------------------------------------- -}

	  if (young_gen()->is_in_reserved(p)) {
	    return true;
	  }
	
	  if (old_gen()->is_in_reserved(p)) {
	    return true;
	  }
	
	  if (perm_gen()->is_in_reserved(p)) {
	    return true;
	  }
	
	  return false;
	}
	
```


