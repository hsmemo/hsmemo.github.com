---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.inline.hpp

### 名前(function name)
```
HeapWord* CollectedHeap::allocate_from_tlab(Thread* thread, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(UseTLAB, "should use UseTLAB");
	
  {- -------------------------------------------
  (1) まずは, 既にカレントスレッドが確保済みの TLAB からメモリを確保してみる.
      成功すればここでリターン.
      ---------------------------------------- -}

	  HeapWord* obj = thread->tlab().allocate(size);
	  if (obj != NULL) {
	    return obj;
	  }

  {- -------------------------------------------
  (1) 失敗した場合は, CollectedHeap::allocate_from_tlab_slow() で新しい TLAB を確保し, 再度メモリ確保を試みる.
      ---------------------------------------- -}

	  // Otherwise...
	  return allocate_from_tlab_slow(thread, size);
	}
	
```


