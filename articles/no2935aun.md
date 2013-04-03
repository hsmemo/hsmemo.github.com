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
  bool doHeapRegion(HeapRegion* hr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 処理対象の HeapRegion に応じて, 以下のどれかの処理を行う.
      
      * 処理対象の HeapRegion が Humongous であり, かつ Humongous オブジェクトの先頭にあたる場合:
        * そのオブジェクトが生きている場合 (= is_gc_marked() が true の場合): 
          oopDesc::forward_to() を呼んで, コンパクション先を指す forwarding pointer を埋める.
          (なお, コンパクション先は現在地点とする)
    
        * そのオブジェクトが死んでいる場合:
          G1PrepareCompactClosure::free_humongous_region() を呼んで, #TODO
  
      * 処理対象の HeapRegion が Humongous だが, オブジェクトの先頭ではない場合:
        何もしない.
  
      * 処理対象の HeapRegion が Humongous ではない場合:
        ContiguousSpace::prepare_for_compaction() を呼んで, HeapRegion 内のオブジェクトに forwarding pointer を埋める.
        (ついでに, コンパクション後にはもう意味がなくなるので ModRefBarrierSet 中の対応箇所のクリアもしている)
      ---------------------------------------- -}

	    if (hr->isHumongous()) {
	      if (hr->startsHumongous()) {
	        oop obj = oop(hr->bottom());
	        if (obj->is_gc_marked()) {
	          obj->forward_to(obj);
	        } else  {
	          free_humongous_region(hr);
	        }
	      } else {
	        assert(hr->continuesHumongous(), "Invalid humongous.");
	      }
	    } else {
	      hr->prepare_for_compaction(&_cp);
	      // Also clear the part of the card table that will be unused after
	      // compaction.
	      _mrbs->clear(MemRegion(hr->compaction_top(), hr->end()));
	    }
	    return false;
	  }
	
```


