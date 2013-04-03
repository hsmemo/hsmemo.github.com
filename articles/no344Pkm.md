---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.inline.hpp

### 名前(function name)
```
inline HeapWord* ThreadLocalAllocBuffer::allocate(size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  invariants();

  {- -------------------------------------------
  (1) もし要求サイズ以上の未使用領域が残っていれば, top をずらして領域を確保し, それをリターンする.
      ---------------------------------------- -}

	  HeapWord* obj = top();
	  if (pointer_delta(end(), obj) >= size) {
	    // successful thread-local allocation
	#ifdef ASSERT
	    // Skip mangling the space corresponding to the object header to
	    // ensure that the returned space is not considered parsable by
	    // any concurrent GC thread.
	    size_t hdr_size = oopDesc::header_size();
	    Copy::fill_to_words(obj + hdr_size, size - hdr_size, badHeapWordVal);
	#endif // ASSERT
	    // This addition is safe because we know that top is
	    // at least size below end, so the add can't wrap.
	    set_top(obj + size);
	
	    invariants();
	    return obj;
	  }

  {- -------------------------------------------
  (1) もし未使用領域が要求サイズ未満なら, NULL をリターン.
      ---------------------------------------- -}

	  return NULL;
	}
	
```


