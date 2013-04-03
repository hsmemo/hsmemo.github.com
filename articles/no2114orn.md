---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
void os::PlatformEvent::park() {           // AKA: down()
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (コメントによると
       os::PlatformEvent::park() を呼び出していいのは
       この os::PlatformEvent オブジェクトに関連付いているスレッドだけ)
      ---------------------------------------- -}

	  // Invariant: Only the thread associated with the Event/PlatformEvent
	  // may call park().

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int v ;

  {- -------------------------------------------
  (1) Atomic::cmpxchg() で _Event フィールドの値を 1デクリメントする.
      (Atomic::cmpxchg() は失敗することもあるが, 成功するまで繰り返す)
      ---------------------------------------- -}

	  for (;;) {
	      v = _Event ;
	      if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ;
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (v >= 0, "invariant") ;

  {- -------------------------------------------
  (1) もし変更前の _Event フィールドの値が 0 だった場合は, 
      os::Solaris::cond_wait() を呼んで
      誰かが unpark() してくれるまで眠りにつく.
  
      (なお, 以下の処理は _mutex で排他された critical section 内で行う)
  
      (間違って起きてしまうこともあるので, 
       目が覚めた後で _Event フィールドもチェックしている.
       _Event フィールドが負値のままであれば, 
       0 以上の値になるまで os::Solaris::cond_wait() を呼び続ける.)
    
      (なお, 眠っているスレッドがいることを unpark() する側に伝えるために, 
       眠りにつく前に _nParked フィールドの値をインクリメントし, 
       unpark() されたら _nParked フィールドの値を戻している.)
  
      (なお, ClearFPUAtPark オプションが指定されている場合は... #TODO)
  
      (unpark() されて待機が解けたら, _Event フィールドの値は 0 にしている)
      ---------------------------------------- -}

	  if (v == 0) {
	     // Do this the hard way by blocking ...
	     // See http://monaco.sfbay/detail.jsf?cr=5094058.
	     // TODO-FIXME: for Solaris SPARC set fprs.FEF=0 prior to parking.
	     // Only for SPARC >= V8PlusA
	#if defined(__sparc) && defined(COMPILER2)
	     if (ClearFPUAtPark) { _mark_fpu_nosave() ; }
	#endif
	     int status = os::Solaris::mutex_lock(_mutex);
	     assert_status(status == 0, status,  "mutex_lock");
	     guarantee (_nParked == 0, "invariant") ;
	     ++ _nParked ;
	     while (_Event < 0) {
	        // for some reason, under 2.7 lwp_cond_wait() may return ETIME ...
	        // Treat this the same as if the wait was interrupted
	        // With usr/lib/lwp going to kernel, always handle ETIME
	        status = os::Solaris::cond_wait(_cond, _mutex);
	        if (status == ETIME) status = EINTR ;
	        assert_status(status == 0 || status == EINTR, status, "cond_wait");
	     }
	     -- _nParked ;
	     _Event = 0 ;
	     status = os::Solaris::mutex_unlock(_mutex);
	     assert_status(status == 0, status, "mutex_unlock");
	  }
	}
	
```


