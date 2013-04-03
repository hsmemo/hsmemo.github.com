---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1OopClosures.inline.hpp

### 名前(function name)
```
template <class T> inline void FilterIntoCSClosure::do_oop_nv(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) p 引数で渡されたポインタが, (NULL ではなくかつ) collection set 内を指していれば,
      コンストラクタ引数で指定された OopClosure (_oc) を 
      そのポインタに適用する.
  
      (なお, FILTERINTOCSCLOSURE_DOHISTOGRAMCOUNT 定数が 0 でない場合には,
       コンストラクタ引数で指定された DirtyCardToOopClosure のカウントアップ処理も行っている)
      ---------------------------------------- -}

	  T heap_oop = oopDesc::load_heap_oop(p);
	  if (!oopDesc::is_null(heap_oop) &&
	      _g1->obj_in_cs(oopDesc::decode_heap_oop_not_null(heap_oop))) {
	    _oc->do_oop(p);
	#if FILTERINTOCSCLOSURE_DOHISTOGRAMCOUNT
	    if (_dcto_cl != NULL)
	      _dcto_cl->incr_count();
	#endif
	  }
	}
	
```


