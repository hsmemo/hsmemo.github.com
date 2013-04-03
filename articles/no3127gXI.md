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
  void add_reference_work(OopOrNarrowOopStar from, bool par) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Must make this robust in case "from" is not in "_hr", because of
	    // concurrency.
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#if HRRS_VERBOSE
	    gclog_or_tty->print_cr("    PRT::Add_reference_work(" PTR_FORMAT "->" PTR_FORMAT").",
	                           from, *from);
	#endif
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    HeapRegion* loc_hr = hr();

  {- -------------------------------------------
  (1) PerRegionTable::add_card_work() を呼んで記録処理を行う.
      (ただし, from 引数の指すアドレスが
       この PerRegionTable が表す HeapRegion 内にない場合は
       (= is_in_reserved() が false であれば),
       記録する必要は無いので, 何もしない)
      ---------------------------------------- -}

	    // If the test below fails, then this table was reused concurrently
	    // with this operation.  This is OK, since the old table was coarsened,
	    // and adding a bit to the new table is never incorrect.
	    if (loc_hr->is_in_reserved(from)) {
	      size_t hw_offset = pointer_delta((HeapWord*)from, loc_hr->bottom());
	      CardIdx_t from_card = (CardIdx_t)
	          hw_offset >> (CardTableModRefBS::card_shift - LogHeapWordSize);
	
	      assert(0 <= from_card && from_card < HeapRegion::CardsPerRegion,
	             "Must be in range.");
	      add_card_work(from_card, par);
	    }
	  }
	
```


