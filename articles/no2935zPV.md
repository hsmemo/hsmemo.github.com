---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionSeq.cpp
### 説明(description)

```
// The first argument r is the heap region at which iteration begins.
// This operation runs fastest when r is NULL, or the heap region for
// which a HeapRegionClosure most recently returned true, or the
// heap region immediately to its right in the sequence.  In all
// other cases a linear search is required to find the index of r.

```

### 名前(function name)
```
void HeapRegionSeq::iterate_from(HeapRegion* r, HeapRegionClosure* blk) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}
	
	  // :::: FIXME ::::
	  // Static cache value is bad, especially when we start doing parallel
	  // remembered set update. For now just don't cache anything (the
	  // code in the def'd out blocks).
	
  {- -------------------------------------------
  (1) (??)
      ---------------------------------------- -}

	#if 0
	  static int cached_j = 0;
	#endif

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int len = _regions.length();
	  int j = 0;

  {- -------------------------------------------
  (1) (??)
    
      (色々コメントアウトされすぎて何もしていないような... #TODO
       r 引数に対応する index を見つけたいんだと思うが...)
      ---------------------------------------- -}

	  // Find the index of r.
	  if (r != NULL) {
	#if 0
	    assert(cached_j >= 0, "Invariant.");
	    if ((cached_j < len) && (r == _regions.at(cached_j))) {
	      j = cached_j;
	    } else if ((cached_j + 1 < len) && (r == _regions.at(cached_j + 1))) {
	      j = cached_j + 1;
	    } else {
	      j = find(r);
	#endif
	      if (j < 0) {
	        j = 0;
	      }
	#if 0
	    }
	#endif
	  }

  {- -------------------------------------------
  (1) 「r 引数に対応する index (= j)」よりも大きな index を持つ全ての HeapRegion に対して, 
      blk 引数で与えられた HeapRegionClosure オブジェクトを適用する.
    
      (もし途中で返値として true が返された場合は, HeapRegionClosure::incomplete() を呼んだ後, その時点でリターン)
      ---------------------------------------- -}

	  int i;
	  for (i = j; i < len; i += 1) {
	    int res = blk->doHeapRegion(_regions.at(i));
	    if (res) {
	#if 0
	      cached_j = i;
	#endif
	      blk->incomplete();
	      return;
	    }
	  }

  {- -------------------------------------------
  (1) 「r 引数に対応する index (= j)」よりも小さな index を持つ全ての HeapRegion に対して, 
      blk 引数で与えられた HeapRegionClosure オブジェクトを適用する.
  
      (もし途中で返値として true が返された場合は, HeapRegionClosure::incomplete() を呼んだ後, その時点でリターン)
      ---------------------------------------- -}

	  for (i = 0; i < j; i += 1) {
	    int res = blk->doHeapRegion(_regions.at(i));
	    if (res) {
	#if 0
	      cached_j = i;
	#endif
	      blk->incomplete();
	      return;
	    }
	  }
	}
	
```


