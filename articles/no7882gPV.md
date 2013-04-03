---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/nmethod.cpp

### 名前(function name)
```
void nmethod::oops_do_marking_prologue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  NOT_PRODUCT(if (TraceScavenge)  tty->print_cr("[oops_do_marking_prologue"));

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_oops_do_mark_nmethods == NULL, "must not call oops_do_marking_prologue twice in a row");

  {- -------------------------------------------
  (1) _oops_do_mark_nmethods フィールドを, NULL から non-NULL な値(NMETHOD_SENTINEL)に書き換える.
      (なお, 他のスレッドからもきちんと見えるようにするため, 普通の store ではなく Atomic::cmpxchg_ptrAtomic() を使っているとのこと)
      ---------------------------------------- -}

	  // We use cmpxchg_ptr instead of regular assignment here because the user
	  // may fork a bunch of threads, and we need them all to see the same state.
	  void* observed = Atomic::cmpxchg_ptr(NMETHOD_SENTINEL, &_oops_do_mark_nmethods, NULL);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(observed == NULL, "no races in this sequential code");
	}
	
```


