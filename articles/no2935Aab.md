---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1MarkSweep.cpp

### 名前(function name)
```
  bool doHeapRegion(HeapRegion* r) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 処理対象の HeapRegion に応じて, 以下のどれかの処理を行う.
      
      * 処理対象の HeapRegion が Humongous であり, かつ Humongous オブジェクトの先頭にあたる場合:
        oopDesc::adjust_pointers() を呼んで, そのオブジェクトのポインタを修正する.
  
      * 処理対象の HeapRegion が Humongous だが, オブジェクトの先頭ではない場合:
        何もしない.
  
      * 処理対象の HeapRegion が Humongous ではない場合:
        CompactibleSpace::adjust_pointers() を呼んで, HeapRegion 内のポインタを修正する.
      ---------------------------------------- -}

	    if (r->isHumongous()) {
	      if (r->startsHumongous()) {
	        // We must adjust the pointers on the single H object.
	        oop obj = oop(r->bottom());
	        debug_only(GenMarkSweep::track_interior_pointers(obj));
	        // point all the oops to the new location
	        obj->adjust_pointers();
	        debug_only(GenMarkSweep::check_interior_pointers());
	      }
	    } else {
	      // This really ought to be "as_CompactibleSpace"...
	      r->adjust_pointers();
	    }
	    return false;
	  }
	
```


