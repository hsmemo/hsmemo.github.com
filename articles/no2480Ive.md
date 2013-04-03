---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/mutableSpace.cpp
### 説明(description)

```
// This version is lock-free.
```

### 名前(function name)
```
HeapWord* MutableSpace::cas_allocate(size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CAS で top を size 分増加させて領域を確保し, それをリターンするだけ.
      (CAS が失敗すれば, 成功するまでループで繰り返す.
       なお, 残りの領域長が size 未満であれば NULL を返す.)
      ---------------------------------------- -}

	  do {
	    HeapWord* obj = top();
	    if (pointer_delta(end(), obj) >= size) {
	      HeapWord* new_top = obj + size;
	      HeapWord* result = (HeapWord*)Atomic::cmpxchg_ptr(new_top, top_addr(), obj);
	      // result can be one of two:
	      //  the old top value: the exchange succeeded
	      //  otherwise: the new value of the top is returned.
	      if (result != obj) {
	        continue; // another thread beat us to the allocation, try again
	      }
	      assert(is_object_aligned((intptr_t)obj) && is_object_aligned((intptr_t)new_top),
	             "checking alignment");
	      return obj;
	    } else {
	      return NULL;
	    }
	  } while (true);
	}
	
```


