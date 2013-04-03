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
template <class T> inline void FilterAndMarkInHeapRegionAndIntoCSClosure::do_oop_nv(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) p 引数で渡されたポインタが, (NULL ではなくかつ) collection set 内を指していれば,
      コンストラクタ引数で指定された OopsInHeapRegionClosure (_oc) を 
      そのポインタに適用する.
    
      一方, p 引数で渡されたポインタが (NULL ではなくかつ) 
      collection set 内も young region も指していない場合,
      ConcurrentMark::grayRoot() を呼び出す.)
      ---------------------------------------- -}

	  T heap_oop = oopDesc::load_heap_oop(p);
	  if (!oopDesc::is_null(heap_oop)) {
	    oop obj = oopDesc::decode_heap_oop_not_null(heap_oop);
	    HeapRegion* hr = _g1->heap_region_containing((HeapWord*) obj);
	    if (hr != NULL) {
	      if (hr->in_collection_set())
	        _oc->do_oop(p);
	      else if (!hr->is_young())
	        _cm->grayRoot(obj);
	    }
	  }
	}
	
```


