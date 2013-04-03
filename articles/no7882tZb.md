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
void nmethod::oops_do_marking_epilogue() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_oops_do_mark_nmethods != NULL, "must not call oops_do_marking_epilogue twice in a row");

  {- -------------------------------------------
  (1) (変数宣言など) (なお _oops_do_mark_nmethods は volatile)
      ---------------------------------------- -}

	  nmethod* cur = _oops_do_mark_nmethods;

  {- -------------------------------------------
  (1) _oops_do_mark_nmethods リストにつながっている全ての nmethod オブジェクトに対して, 以下の処理を行う.
      * _oops_do_mark_link フィールドを NULL に戻す
      * nmethod::fix_oop_relocations() を呼んで, oop のアドレスに依存している箇所を relocation する.
  
      (ついでに, (トレース出力)も出している)
      ---------------------------------------- -}

	  while (cur != NMETHOD_SENTINEL) {
	    assert(cur != NULL, "not NULL-terminated");
	    nmethod* next = cur->_oops_do_mark_link;
	    cur->_oops_do_mark_link = NULL;
	    cur->fix_oop_relocations();
	    NOT_PRODUCT(if (TraceScavenge)  cur->print_on(tty, "oops_do, unmark"));
	    cur = next;
	  }

  {- -------------------------------------------
  (1) (変数宣言など) (なお _oops_do_mark_nmethods は volatile)
      ---------------------------------------- -}

	  void* required = _oops_do_mark_nmethods;

  {- -------------------------------------------
  (1) _oops_do_mark_nmethods フィールドを, non-NULL な値(NMETHOD_SENTINEL)から NULL に戻す.
      (なお, 他のスレッドからもきちんと見えるようにするため, 普通の store ではなく Atomic::cmpxchg_ptrAtomic() を使っている模様)
      ---------------------------------------- -}

	  void* observed = Atomic::cmpxchg_ptr(NULL, &_oops_do_mark_nmethods, required);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(observed == required, "no races in this sequential code");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  NOT_PRODUCT(if (TraceScavenge)  tty->print_cr("oops_do_marking_epilogue]"));
	}
	
```


