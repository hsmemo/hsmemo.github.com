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
        HeapRegion::reset_during_compaction() を呼んで, #TODO
    
        また, その Humongous オブジェクトが生きている場合 (= is_gc_marked() が true の場合) は, 
        oopDesc::init_mark() を呼んで, mark フィールドを初期状態の値に戻す作業も行っている.
  
      * 処理対象の HeapRegion が Humongous だが, オブジェクトの先頭ではない場合:
        何もしない.
  
      * 処理対象の HeapRegion が Humongous ではない場合:
        CompactibleSpace::compact() を呼んで, HeapRegion 内のオブジェクトを移動させる.
      ---------------------------------------- -}

	    if (hr->isHumongous()) {
	      if (hr->startsHumongous()) {
	        oop obj = oop(hr->bottom());
	        if (obj->is_gc_marked()) {
	          obj->init_mark();
	        } else {
	          assert(hr->is_empty(), "Should have been cleared in phase 2.");
	        }
	        hr->reset_during_compaction();
	      }
	    } else {
	      hr->compact();
	    }
	    return false;
	  }
	
```


