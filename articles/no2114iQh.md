---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/genOopClosures.inline.hpp

### 名前(function name)
```
template <class T> inline void OopsInGenClosure::do_barrier(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(generation()->is_in_reserved(p), "expected ref in generation");

  {- -------------------------------------------
  (1) もし処理対象のポインタの指し先(obj)が
      コンストラクタ引数で指定された境界値(_gen_boundary)より小さければ, 
      CardTableRS::inline_write_ref_field_gc() を呼んで
      該当する card を youngergen_card (つまり「New 領域への参照有り」) に設定しておく.
      ---------------------------------------- -}

	  T heap_oop = oopDesc::load_heap_oop(p);
	  assert(!oopDesc::is_null(heap_oop), "expected non-null oop");
	  oop obj = oopDesc::decode_heap_oop_not_null(heap_oop);
	  // If p points to a younger generation, mark the card.
	  if ((HeapWord*)obj < _gen_boundary) {
	    _rs->inline_write_ref_field_gc(p, obj);
	  }
	}
	
```


