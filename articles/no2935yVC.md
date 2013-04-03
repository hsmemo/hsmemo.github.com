---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
void G1CollectedHeap::do_full_collection(bool clear_all_soft_refs) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) G1CollectedHeap::do_collection() を呼び出すだけ.
      ---------------------------------------- -}

	  // do_collection() will return whether it succeeded in performing
	  // the GC. Currently, there is no facility on the
	  // do_full_collection() API to notify the caller than the collection
	  // did not succeed (e.g., because it was locked out by the GC
	  // locker). So, right now, we'll ignore the return value.
	  bool dummy = do_collection(true,                /* explicit_gc */
	                             clear_all_soft_refs,
	                             0                    /* word_size */);
	}
	
```


