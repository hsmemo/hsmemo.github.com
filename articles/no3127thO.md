---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegionRemSet.cpp

### 名前(function name)
```
  void add_card_work(CardIdx_t from_card, bool par) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下のどれかの方法でビットを立てる (もしくは何もしない).
  
      * 既にマーク済み(= at() が true)であれば, 何もしない
      * par 引数が true であれば, par_at_put() でビットを立てる.
      * par 引数が false であれば, at_put() でビットを立てる.
  
      (ついでに, デバッグパスでは _occupied を Atomic::inc してたりするが...)
      ---------------------------------------- -}

	    if (!_bm.at(from_card)) {
	      if (par) {
	        if (_bm.par_at_put(from_card, 1)) {
	#if PRT_COUNT_OCCUPIED
	          Atomic::inc(&_occupied);
	#endif
	        }
	      } else {
	        _bm.at_put(from_card, 1);
	#if PRT_COUNT_OCCUPIED
	        _occupied++;
	#endif
	      }
	    }
	  }
	
```


