---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/markSweep.inline.hpp

### 名前(function name)
```
template <class T> inline void MarkSweep::mark_and_push(T* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (処理対象のオブジェクトが NULL, もしくは既にマークされている場合には, 何も行わない)
  
      処理対象のオブジェクトが NULL ではなく未だマークもされていなければ, 
      MarkSweep::mark_object() でマークを付加し,
      Stack::push() で marking stack にプッシュする.
      ---------------------------------------- -}

	//  assert(Universe::heap()->is_in_reserved(p), "should be in object space");
	  T heap_oop = oopDesc::load_heap_oop(p);
	  if (!oopDesc::is_null(heap_oop)) {
	    oop obj = oopDesc::decode_heap_oop_not_null(heap_oop);
	    if (!obj->mark()->is_marked()) {
	      mark_object(obj);
	      _marking_stack.push(obj);
	    }
	  }
	}
	
```


