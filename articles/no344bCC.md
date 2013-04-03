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
void CollectedHeap::init_obj(HeapWord* obj, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  assert(obj != NULL, "cannot initialize NULL object");
	  const size_t hs = oopDesc::header_size();
	  assert(size >= hs, "unexpected object size");

  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  ((oop)obj)->set_klass_gap(0);

  {- -------------------------------------------
  (1) Copy::fill_to_aligned_words() で, ヘッダー部を除いた部分を 0 クリアする 
      ---------------------------------------- -}

	  Copy::fill_to_aligned_words(obj + hs, size - hs);
	}
	
```


