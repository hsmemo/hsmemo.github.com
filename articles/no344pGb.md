---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp
### 説明(description)
現在の TLAB の残り領域をダミーの配列で埋めて, 
全てオブジェクトで埋まっている(ように見える)状態にする
(途中に空き領域があると GC 時にヒープ中を辿る際などに問題になるため).

また, 引数の値によっては, 追加措置として TLAB の retire 処理も行う.

なお, これによって無駄になる領域についての統計情報は
この関数では収集していないので, これを呼び出す側でやるようにとのこと.
(ThreadLocalAllocBuffer::clear_before_allocation() 等を参照, とのこと)

```
// Fills the current tlab with a dummy filler array to create
// an illusion of a contiguous Eden and optionally retires the tlab.
// Waste accounting should be done in caller as appropriate; see,
// for example, clear_before_allocation().
```

### 名前(function name)
```
void ThreadLocalAllocBuffer::make_parsable(bool retire) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ?? (逆に end() が NULL のケースとは?? 未初期化状態の ThreadLocalAllocBuffer?? 既に make_parsable() 済みの TLAB?? #TODO)
      ---------------------------------------- -}

	  if (end() != NULL) {

  {- -------------------------------------------
  (1) (assert)
      (See: ThreadLocalAllocBuffer::invariants())
      ---------------------------------------- -}

	    invariants();
	
  {- -------------------------------------------
  (1) 引数で retire 処理が要求されていた場合には, 
      Thread::incr_allocated_bytes() を呼び出して
      確保量に関する統計情報の更新処理を行っておく 
      (See: [here](no2114Q4Z.html) for details))
      ---------------------------------------- -}

	    if (retire) {
	      myThread()->incr_allocated_bytes(used_bytes());
	    }
	
  {- -------------------------------------------
  (1) CollectedHeap::fill_with_object() で, TLAB 内の残り領域を dummy の配列で埋める.
      ---------------------------------------- -}

	    CollectedHeap::fill_with_object(top(), hard_end(), retire);
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	    if (retire || ZeroTLAB) {  // "Reset" the TLAB
	      set_start(NULL);
	      set_top(NULL);
	      set_pf_top(NULL);
	      set_end(NULL);
	    }
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!(retire || ZeroTLAB)  ||
	         (start() == NULL && end() == NULL && top() == NULL),
	         "TLAB must be reset");
	}
	
```


