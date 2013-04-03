---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.hpp

### 名前(function name)
```
  bool parMark(HeapWord* addr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(_bmStartWord <= addr && addr < (_bmStartWord + _bmWordSize),
	           "outside underlying space?");

  {- -------------------------------------------
  (1) BitMap::par_set_bit() でビットを立て, 結果をリターン
      ---------------------------------------- -}

	    return _bm.par_at_put(heapWordToOffset(addr), true);
	  }
	
```


