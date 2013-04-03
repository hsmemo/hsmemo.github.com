---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.inline.hpp
### 説明(description)

```
// Because of the requirement of keeping "_offsets" up to date with the
// allocations, we sequentialize these with a lock.  Therefore, best if
// this is used for larger LAB allocations only.
```

### 名前(function name)
```
inline HeapWord* G1OffsetTableContigSpace::par_allocate(size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, 以下の処理は _par_alloc_lock に格納している Mutex で排他を取って行う)
      (これは, 上記のコメントの通り, block offset table の更新と atomic に行う必要があるため)
      ---------------------------------------- -}

	  MutexLocker x(&_par_alloc_lock);

  {- -------------------------------------------
  (1) ContiguousSpace::allocate() で確保を行い, 結果をリターン.
      (なお, 確保が成功した場合には _offset に格納している block offset table を更新している)
      (なお, lock を取っているので ContiguousSpace::par_allocate() の代わりに ContiguousSpace::allocate() を使っているとのこと)
      ---------------------------------------- -}

	  // Given that we take the lock no need to use par_allocate() here.
	  HeapWord* res = ContiguousSpace::allocate(size);
	  if (res != NULL) {
	    _offsets.alloc_block(res, size);
	  }
	  return res;
	}
	
```


