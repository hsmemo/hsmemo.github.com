---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/virtualspace.cpp

### 名前(function name)
```
char* ReservedSpace::reserve_and_align(const size_t reserve_size,
                                       const size_t prefix_size,
                                       const size_t prefix_align,
                                       const size_t suffix_size,
                                       const size_t suffix_align)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(reserve_size > prefix_size + suffix_size, "should not be here");
	
  {- -------------------------------------------
  (1) os::reserve_memory() でメモリ領域を確保し, 
      ReservedSpace::align_reserved_region() でアラインさせた結果を取得.
    
      (なお, os::reserve_memory() が失敗すれば, ここで NULL をリターン)
      ---------------------------------------- -}

	  char* raw_addr = os::reserve_memory(reserve_size, NULL, prefix_align);
	  if (raw_addr == NULL) return NULL;
	
	  char* result = align_reserved_region(raw_addr, reserve_size, prefix_size,
	                                       prefix_align, suffix_size,
	                                       suffix_align);
	  if (result == NULL && !os::release_memory(raw_addr, reserve_size)) {
	    fatal("os::release_memory failed");
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  if (result != NULL) {
	    const size_t raw = size_t(raw_addr);
	    const size_t res = size_t(result);
	    assert(res >= raw, "alignment decreased start addr");
	    assert(res + prefix_size + suffix_size <= raw + reserve_size,
	           "alignment increased end addr");
	    assert((res & prefix_align - 1) == 0, "bad alignment of prefix");
	    assert((res + prefix_size & suffix_align - 1) == 0,
	           "bad alignment of suffix");
	  }
	#endif
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return result;
	}
	
```


