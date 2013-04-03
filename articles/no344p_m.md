---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp


### 本体部(body)
```
	void ThreadLocalAllocBuffer::clear_before_allocation() {

  {- -------------------------------------------
  (1) (プロファイル情報の記録) 
      (ここでは, 未使用のまま捨てられてしまう領域長(_slow_refill_waste)をインクリメント)
      (See: GlobalTLABStats, ThreadLocalAllocBuffer::print_stats())
      ---------------------------------------- -}

	  _slow_refill_waste += (unsigned)remaining();

  {- -------------------------------------------
  (1) ThreadLocalAllocBuffer::make_parsable() で
      現在の TLAB の残りのスペースを dummy の int 配列で埋めておく, 
      ---------------------------------------- -}

	  make_parsable(true);   // also retire the TLAB
	}
	
```


