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
template <class T> inline void FilterOutOfRegionClosure::do_oop_nv(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) p 引数で渡されたポインタが, (NULL ではなくかつ) 
      指定されたアドレス範囲の外を差していれば (= bottomよりも下, あるいは end 以上であれば)
      コンストラクタ引数で指定された OopClosure (_oc) を 
      そのポインタに適用する.
  
      (なお, FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT 定数が 0 でない場合には,
       OopClosure を適用した回数を _out_of_region という int フィールドに記録している)
      ---------------------------------------- -}

	  T heap_oop = oopDesc::load_heap_oop(p);
	  if (!oopDesc::is_null(heap_oop)) {
	    HeapWord* obj_hw = (HeapWord*)oopDesc::decode_heap_oop_not_null(heap_oop);
	    if (obj_hw < _r_bottom || obj_hw >= _r_end) {
	      _oc->do_oop(p);
	#if FILTEROUTOFREGIONCLOSURE_DOHISTOGRAMCOUNT
	      _out_of_region++;
	#endif
	    }
	  }
	}
	
```


