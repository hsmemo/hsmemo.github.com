---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp

### 名前(function name)
```
  inline HeapWord* par_allocate_no_bot_updates(size_t word_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(is_young(), "we can only skip BOT updates on young regions");

  {- -------------------------------------------
  (1) ContiguousSpace::par_allocate() で確保を行い, 結果をリターン.
      ---------------------------------------- -}

	    return ContiguousSpace::par_allocate(word_size);
	  }
	
```


