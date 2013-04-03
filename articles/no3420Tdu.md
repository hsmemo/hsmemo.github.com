---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp

### 名前(function name)
```
void CountNonCleanMemRegionClosure::do_MemRegion(MemRegion mr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) mr 引数の MemRegion が G1CollectedHeap 内のメモリ領域であれば, 
      それに対応する card の個数分だけ _n フィールドを増加させる.
    
      (また最初に呼ばれた際には, _start_first フィールドにその MemRegion の先頭アドレスを記録している)
      ---------------------------------------- -}

	  if (_g1->is_in_g1_reserved(mr.start())) {
	    _n += (int) ((mr.byte_size() / CardTableModRefBS::card_size));
	    if (_start_first == NULL) _start_first = mr.start();
	  }
	}
	
```


