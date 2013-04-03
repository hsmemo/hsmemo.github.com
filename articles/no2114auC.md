---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/park.cpp

### 名前(function name)
```
void ParkEvent::Release (ParkEvent * ev) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数が NULL だった場合は, 何もしない(ここでリターン).
      ---------------------------------------- -}

	  if (ev == NULL) return ;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (ev->FreeNext == NULL      , "invariant") ;

  {- -------------------------------------------
  (1) 引数の ParkEvent オブジェクトを FreeList につなぐ.
      (他のスレッドが同時に処理していると失敗することもあるので, 成功するまで Atomic::cmpxchg_ptr() を繰り返す)
      ---------------------------------------- -}

	  ev->AssociatedWith = NULL ;
	  for (;;) {
	    // Push ev onto FreeList
	    // The mechanism is "half" lock-free.
	    ParkEvent * List = FreeList ;
	    ev->FreeNext = List ;
	    if (Atomic::cmpxchg_ptr (ev, &FreeList, List) == List) break ;
	  }
	}
	
```


