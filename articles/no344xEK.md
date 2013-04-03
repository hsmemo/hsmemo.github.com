---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_interface/collectedHeap.cpp

### 名前(function name)
```
void CollectedHeap::pre_initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) C2 JIT Compiler 用の処理. 
      _defer_initial_card_mark フィールドを初期化する (#TODO).
      ---------------------------------------- -}

	  // Used for ReduceInitialCardMarks (when COMPILER2 is used);
	  // otherwise remains unused.
	#ifdef COMPILER2
	  _defer_initial_card_mark =    ReduceInitialCardMarks && can_elide_tlab_store_barriers()
	                             && (DeferInitialCardMark || card_mark_must_follow_store());
	#else
	  assert(_defer_initial_card_mark == false, "Who would set it?");
	#endif
	}
	
```


