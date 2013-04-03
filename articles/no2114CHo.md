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
int os::PlatformEvent::park(jlong millis) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (_nParked == 0, "invariant") ;

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
  (1) もし変更前の _Event フィールドの値が 
      0 以外だった場合は (つまりは 1 だった場合は), 
      ここでリターン
      ---------------------------------------- -}

	  if (v != 0) return OS_OK ;
	
  {- -------------------------------------------
  (1) 変更前の _Event フィールドの値が 0 だった場合は, 
      以下で os::Solaris::cond_timedwait() を呼んで
      誰かが unpark() してくれるかタイムアウト時間が経過するまで眠りにつく.
  
      (なお, 以下の処理は _mutex で排他された critical section 内で行う)
  
      (間違って起きてしまうこともあるので, 
       目が覚めた後で以下の値をチェックしている.
       * os::Solaris::cond_timedwait() の返値
         ETIME か ETIMEDOUT であれば, 実際にタイムアウト時間が経過したということなので, 待機は終了.
       * _Event フィールドの値
         0 以上であれば, unpark() されたということなので, 待機は終了.
  
       起きた時点で以上の条件がどちらも満たされていなければ, 
       どちらかが満たされるようになるまで os::Solaris::cond_timedwait() を呼び続ける.
       
       ただし, FilterSpuriousWakeups オプションが指定されていない場合には, 
       間違って起きてしまった場合でも, とにかく一度起きてしまったら待機処理を終了する (これが以前の挙動らしい))
    
      (なお, 眠っているスレッドがいることを unpark() する側に伝えるために, 
       眠りにつく前に _nParked フィールドの値をインクリメントし, 
       unpark() されたら _nParked フィールドの値を戻している.)
    
      (なお, ClearFPUAtPark オプションが指定されている場合は... #TODO)
  
      (unpark() されて待機が解けたら, _Event フィールドの値は 0 にしている.
    
       なお返値としては, このクリア処理の直前に _Event が 0 以上であれば OS_OK が返され, 
       それ以外なら OS_TIMEOUT が返される.
       というわけで, タイムアウトと unpark() が同時の場合は unpark() が優先.)
      ---------------------------------------- -}

	  int ret = OS_TIMEOUT;
	  timestruc_t abst;
	  compute_abstime (&abst, millis);
	
	  // See http://monaco.sfbay/detail.jsf?cr=5094058.
	  // For Solaris SPARC set fprs.FEF=0 prior to parking.
	  // Only for SPARC >= V8PlusA
	#if defined(__sparc) && defined(COMPILER2)
	 if (ClearFPUAtPark) { _mark_fpu_nosave() ; }
	#endif
	  int status = os::Solaris::mutex_lock(_mutex);
	  assert_status(status == 0, status, "mutex_lock");
	  guarantee (_nParked == 0, "invariant") ;
	  ++ _nParked ;
	  while (_Event < 0) {
	     int status = os::Solaris::cond_timedwait(_cond, _mutex, &abst);
	     assert_status(status == 0 || status == EINTR ||
	                   status == ETIME || status == ETIMEDOUT,
	                   status, "cond_timedwait");
	     if (!FilterSpuriousWakeups) break ;                // previous semantics
	     if (status == ETIME || status == ETIMEDOUT) break ;
	     // We consume and ignore EINTR and spurious wakeups.
	  }
	  -- _nParked ;
	  if (_Event >= 0) ret = OS_OK ;
	  _Event = 0 ;
	  status = os::Solaris::mutex_unlock(_mutex);
	  assert_status(status == 0, status, "mutex_unlock");
	  return ret;
	}
	
```


