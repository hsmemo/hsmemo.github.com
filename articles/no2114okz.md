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
void Parker::Release (Parker * p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数が NULL だった場合は, 何もしない(ここでリターン).
      ---------------------------------------- -}

	  if (p == NULL) return ;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (p->AssociatedWith != NULL, "invariant") ;
	  guarantee (p->FreeNext == NULL      , "invariant") ;

  {- -------------------------------------------
  (1) 引数の Parker オブジェクトを FreeList につなぐ.
      (他のスレッドが同時に処理していると失敗することもあるので, 成功するまで Atomic::cmpxchg_ptr() を繰り返す)
      ---------------------------------------- -}

	  p->AssociatedWith = NULL ;
	  for (;;) {
	    // Push p onto FreeList
	    Parker * List = FreeList ;
	    p->FreeNext = List ;
	    if (Atomic::cmpxchg_ptr (p, &FreeList, List) == List) break ;
	  }
	}
	
```


