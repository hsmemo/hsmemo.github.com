---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/satbQueue.cpp

### 名前(function name)
```
void SATBMarkQueueSet::par_iterate_closure_all_threads(int worker) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  SharedHeap* sh = SharedHeap::heap();
	  int parity = sh->strong_roots_parity();
	
  {- -------------------------------------------
  (1) 全ての JavaThread に対して, 以下の処理を行う.
  
      Thread::claim_oops_do() を呼び出し, 返値を確認する.
      true が返された場合は, (その JavaThread についてはカレントスレッドが処理を行うということなので)
      ObjPtrQueue::apply_closure() を呼び出して, その JavaThread の satb_mark_queue を処理する.
      (逆に false が返されたら, 他のスレッドが処理してくれるということなので, 何もせずに次の JavaThread の処理に移る)
      ---------------------------------------- -}

	  for(JavaThread* t = Threads::first(); t; t = t->next()) {
	    if (t->claim_oops_do(true, parity)) {
	      t->satb_mark_queue().apply_closure(_par_closures[worker]);
	    }
	  }

  {- -------------------------------------------
  (1) もしカレントスレッドが worker 0 であれば (= worker 引数が 0 であれば), 
      ObjPtrQueue::apply_closure() を呼び出して, shared_satb_queue を処理しておく.
      ---------------------------------------- -}

	  // We'll have worker 0 do this one.
	  if (worker == 0) {
	    shared_satb_queue()->apply_closure(_par_closures[0]);
	  }
	}
	
```


