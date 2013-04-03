---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/universe.cpp

### 名前(function name)
```
void universe2_init() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  EXCEPTION_MARK;

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  Universe::genesis(CATCH);

  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	  // Although we'd like to verify here that the state of the heap
	  // is good, we can't because the main thread has not yet added
	  // itself to the threads list (so, using current interfaces
	  // we can't "fill" its TLAB), unless TLABs are disabled.
	  if (VerifyBeforeGC && !UseTLAB &&
	      Universe::heap()->total_collections() >= VerifyGCStartAt) {
	     Universe::heap()->prepare_for_verify();
	     Universe::verify();   // make sure we're starting with a clean slate
	  }
	}
	
```


