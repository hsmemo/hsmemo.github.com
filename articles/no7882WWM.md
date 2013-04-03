---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void Threads::possibly_parallel_oops_do(OopClosure* f, CodeBlobClosure* cf) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この処理には, マルチスレッドを用いて並列化する機構が入っている.
       オーバーヘッドは小さいはずなので, シングルスレッド時にもその機構はそのまま使っている)
      ---------------------------------------- -}

	  // Introduce a mechanism allowing parallel threads to claim threads as
	  // root groups.  Overhead should be small enough to use all the time,
	  // even in sequential code.

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  SharedHeap* sh = SharedHeap::heap();
	  bool is_par = (sh->n_par_threads() > 0);
	  int cp = SharedHeap::heap()->strong_roots_parity();

  {- -------------------------------------------
  (1) 全ての JavaThread に対して, 以下の処理を行う.
  
      Thread::claim_oops_do() を呼び出し, 返値を確認する.
      true が返された場合は, (その JavaThread についてはカレントスレッドが処理を行うということなので)
      さらに JavaThread::oops_do() の呼び出しも行う.
      (逆に false が返されたら, 他のスレッドが処理してくれるということなので, 何もせずに次の JavaThread の処理に移る)
      ---------------------------------------- -}

	  ALL_JAVA_THREADS(p) {
	    if (p->claim_oops_do(is_par, cp)) {
	      p->oops_do(f, cf);
	    }
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  VMThread* vmt = VMThread::vm_thread();

  {- -------------------------------------------
  (1) VMThread に対しても, (JavaThread と同様に) Thread::claim_oops_do() を呼び出す.
      true が返された場合は, さらに VMThread::oops_do() の呼び出しも行う.
      ---------------------------------------- -}

	  if (vmt->claim_oops_do(is_par, cp))
	    vmt->oops_do(f, cf);
	}
	
```


