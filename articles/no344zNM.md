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
char *
ReservedSpace::align_reserved_region(char* addr, const size_t len,
                                     const size_t prefix_size,
                                     const size_t prefix_align,
                                     const size_t suffix_size,
                                     const size_t suffix_align)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 指定された領域内から条件に合う開始アドレスを計算する.
        (正確には先頭から数えた開始位置のオフセットを計算 (以下の beg_delta))
      ---------------------------------------- -}

	  assert(addr != NULL, "sanity");
	  const size_t required_size = prefix_size + suffix_size;
	  assert(len >= required_size, "len too small");
	
	  const size_t s = size_t(addr);
	  const size_t beg_ofs = s + prefix_size & suffix_align - 1;
	  const size_t beg_delta = beg_ofs == 0 ? 0 : suffix_align - beg_ofs;
	
	  if (len < beg_delta + required_size) {
	     return NULL; // Cannot do proper alignment.
	  }

  {- -------------------------------------------
  (1) その開始アドレスから required_size 分だけを残して
      残りの領域は os::release_memory() で OS に返却し, 
      開始アドレスをリターンする.
      ---------------------------------------- -}

	  const size_t end_delta = len - (beg_delta + required_size);
	
	  if (beg_delta != 0) {
	    os::release_memory(addr, beg_delta);
	  }
	
	  if (end_delta != 0) {
	    char* release_addr = (char*) (s + beg_delta + required_size);
	    os::release_memory(release_addr, end_delta);
	  }
	
	  return (char*) (s + beg_delta);
	}
	
```


