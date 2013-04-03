---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.inline.hpp
### 説明(description)

```
// Attempt to "claim" oop at p via CAS, push the new obj if successful
// This version tests the oop* to make sure it is within the heap before
// attempting marking.
```

### 名前(function name)
```
template <class T>
inline void PSScavenge::copy_and_push_safe_barrier(PSPromotionManager* pm,
                                                   T*                  p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(should_scavenge(p, true), "revisiting object?");
	
  {- -------------------------------------------
  (1) 処理対象のポインタ(p)が指している oop を取得する.
      ---------------------------------------- -}

	  oop o = oopDesc::load_decode_heap_oop_not_null(p);

  {- -------------------------------------------
  (1) ポインタの指し先を PSPromotionManager::copy_to_survivor_space() でコピーする.
      (ただし, 既にコピー済み(= is_forwarded() が true) であればそのコピー先を取得するだけ)
      ---------------------------------------- -}

	  oop new_obj = o->is_forwarded()
	        ? o->forwardee()
	        : pm->copy_to_survivor_space(o);

  {- -------------------------------------------
  (1) 処理対象のポインタ(p)をコピー先のアドレスに書き換える.
      ---------------------------------------- -}

	  oopDesc::encode_store_heap_oop_not_null(p, new_obj);
	
  {- -------------------------------------------
  (1) もし世代をまたがるポインタになっていれば card table に記録しておく.
  
      (より具体的に言うと, 
       もし処理対象のポインタ(p)が Old 領域か Perm 領域内に有り, かつ 
       p の指している先が New 領域内であれば, 
       CardTableExtension::inline_write_ref_field_gc() で
       該当する card table の値を youngergen_card にしておく.)
      ---------------------------------------- -}

	  // We cannot mark without test, as some code passes us pointers
	  // that are outside the heap.
	  if ((!PSScavenge::is_obj_in_young((HeapWord*)p)) &&
	      Universe::heap()->is_in_reserved(p)) {
	    if (PSScavenge::is_obj_in_young((HeapWord*)new_obj)) {
	      card_table()->inline_write_ref_field_gc(p, new_obj);
	    }
	  }
	}
	
```


