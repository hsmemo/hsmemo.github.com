---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/nmethod.cpp
### 説明(description)

```
// An nmethod is "marked" if its _mark_link is set non-null.
// Even if it is the end of the linked list, it will have a non-null link value,
// as long as it is on the list.
// This code must be MP safe, because it is used from parallel GC passes.
```

### 名前(function name)
```
bool nmethod::test_set_oops_do_mark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(nmethod::oops_do_marking_is_active(), "oops_do_marking_prologue must be called");

  {- -------------------------------------------
  (1) (変数宣言など) (なお _oops_do_mark_link は volatile)
      ---------------------------------------- -}

	  nmethod* observed_mark_link = _oops_do_mark_link;

  {- -------------------------------------------
  (1) 以下でこの nmethod オブジェクトの mark 処理を行う.
      処理は, カレントスレッドが _oops_do_mark_link の書き換えに成功するかどうかで 2通りに分かれる.
  
      * カレントスレッドが _oops_do_mark_link の書き換えを行った場合
        (= この時点でまだ _oops_do_mark_link フィールドが NULL であり, かつ 
        _oops_do_mark_link フィールドの書き換えにも成功した場合)
    
        => この nmethod オブジェクトを _oops_do_mark_nmethods リストにつなぎ, false をリターン.
  
      * 他のスレッドによって _oops_do_mark_link が書き換えられた場合 
        (= この時点で _oops_do_mark_link フィールドが NULL ではない場合 or 
         _oops_do_mark_link フィールドの書き換えに失敗した場合)
     
        => 何もせずに true をリターン.
  
      ---------------------------------------- -}

	  if (observed_mark_link == NULL) {

    {- -------------------------------------------
  (1.1) (以下は, この時点でまだ _oops_do_mark_link フィールドが NULL の場合)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) Atomic::cmpxchg_ptr() で _oops_do_mark_link を non-null な値(NMETHOD_SENTINEL)に書き換える.
  
        (なお, 書き換えが失敗した場合は, 他のスレッドが先に書き換えたということなので, 
         残りの処理はそのスレッドに任せればいい.
         このままフォールスルーして true をリターンするだけ.)
        ---------------------------------------- -}

	    // Claim this nmethod for this thread to mark.
	    observed_mark_link = (nmethod*)
	      Atomic::cmpxchg_ptr(NMETHOD_SENTINEL, &_oops_do_mark_link, NULL);
	    if (observed_mark_link == NULL) {

      {- -------------------------------------------
  (1.1.1) (以下は, カレントスレッドが _oops_do_mark_link の書き換えを行った場合)
          ---------------------------------------- -}
	
      {- -------------------------------------------
  (1.1.1) この nmethod オブジェクトを, _oops_do_mark_nmethods フィールドに格納されているリストの先頭に追加する.
          他のスレッドと競合する恐れがあるので, 成功するまで以下の for ループで Atomic::cmpxchg_ptr() を繰り返す.
          追加が完了したら false をリターン.
          ---------------------------------------- -}

	      // Atomically append this nmethod (now claimed) to the head of the list:
	      nmethod* observed_mark_nmethods = _oops_do_mark_nmethods;
	      for (;;) {
	        nmethod* required_mark_nmethods = observed_mark_nmethods;
	        _oops_do_mark_link = required_mark_nmethods;
	        observed_mark_nmethods = (nmethod*)
	          Atomic::cmpxchg_ptr(this, &_oops_do_mark_nmethods, required_mark_nmethods);
	        if (observed_mark_nmethods == required_mark_nmethods)
	          break;
	      }
	      // Mark was clear when we first saw this guy.
	      NOT_PRODUCT(if (TraceScavenge)  print_on(tty, "oops_do, mark"));
	      return false;
	    }
	  }

  {- -------------------------------------------
  (1) (以下は, 他のスレッドによって _oops_do_mark_link が書き換えられた場合
       = _oops_do_mark_link フィールドが NULL ではなかった場合 or Atomic::cmpxchg_ptr() に失敗した場合)
  
      true をリターンするだけ.
      ---------------------------------------- -}

	  // On fall through, another racing thread marked this nmethod before we did.
	  return true;
	}
	
```


