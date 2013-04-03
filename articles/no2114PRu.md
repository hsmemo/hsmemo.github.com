---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
int os::PlatformEvent::park (jlong Millis) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    guarantee (_ParkHandle != NULL , "Invariant") ;
	    guarantee (Millis > 0          , "Invariant") ;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    int v ;
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // CONSIDER: defer assigning a CreateEvent() handle to the Event until
	    // the initial park() operation.
	
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

	    guarantee ((v == 0) || (v == 1), "invariant") ;

  {- -------------------------------------------
  (1) もし変更前の _Event フィールドの値が 
      0 以外だった場合は (つまりは 1 だった場合は), 
      ここでリターン
      ---------------------------------------- -}

	    if (v != 0) return OS_OK ;
	
  {- -------------------------------------------
  (1) 変更前の _Event フィールドの値が 0 だった場合は, 
      以下で WaitForSingleObject() を呼んで
      誰かが unpark() してくれるかタイムアウト時間が経過するまで眠りにつく.
  
      (なお, コメントによると, こういう実装方針になっているとのこと.
         「Windows のバージョンによっては, WaitForSingleObject() に
           長すぎるタイムアウト時間を指定すると問題が起こるとの説(迷信?)がある.
           というわけで, ここではタイムアウト時間を複数に分割し, 小刻みに待機をさせている.
           具体的には, 一度に最大でも 0x10000000 (以下の MAXTIMEOUT) までしか待機させない.
           
           現状では, 経過した時間は WaitForSingleObject() の返値を頼りに計算しているが, 
           spurious wakeup が起こるようなことがあれば 
           os::javaTimeNanos() でまじめに経過時間を調べるようにした方がいいかもしれない.」)
  
      (間違って起きてしまうこともあるので(?), 
       目が覚めた後で以下の値をチェックしている.
       * 引数である Millis の値
         毎回 WaitForSingleObject() から起きる度に MAXTIMEOUT 分だけ減らしていく. 
         (ただし, 割り込まれた場合 (返値が WAIT_OBJECT_0 の場合) には減らさない)
         これが 0 以下になったら, タイムアウトということなので, 待機は終了.
       * _Event フィールドの値
         0 以上であれば, unpark() されたということなので, 待機は終了.
  
       起きた時点で以上の条件がどちらも満たされていなければ, 
       どちらかが満たされるようになるまで WaitForSingleObject() を呼び続ける.)
    
      (unpark() されて待機が解けたら, _Event フィールドの値は 0 にしている.
  
       ついでに, この _Event フィールドのクリア処理が
       この後に行われる load/store に追い抜かれないよう, 
       OrderAccess::fence() も張っている.
  
       なお返値としては, このクリア処理の直前に _Event が 0 以上であれば OS_OK が返され, 
       それ以外なら OS_TIMEOUT が返される.
       というわけで, タイムアウトと unpark() が同時の場合は unpark() が優先.
       とはいえ, これは実装者の好みで変更してもよいとのこと.)
      ---------------------------------------- -}

	    // Do this the hard way by blocking ...
	    // TODO: consider a brief spin here, gated on the success of recent
	    // spin attempts by this thread.
	    //
	    // We decompose long timeouts into series of shorter timed waits.
	    // Evidently large timo values passed in WaitForSingleObject() are problematic on some
	    // versions of Windows.  See EventWait() for details.  This may be superstition.  Or not.
	    // We trust the WAIT_TIMEOUT indication and don't track the elapsed wait time
	    // with os::javaTimeNanos().  Furthermore, we assume that spurious returns from
	    // ::WaitForSingleObject() caused by latent ::setEvent() operations will tend
	    // to happen early in the wait interval.  Specifically, after a spurious wakeup (rv ==
	    // WAIT_OBJECT_0 but _Event is still < 0) we don't bother to recompute Millis to compensate
	    // for the already waited time.  This policy does not admit any new outcomes.
	    // In the future, however, we might want to track the accumulated wait time and
	    // adjust Millis accordingly if we encounter a spurious wakeup.
	
	    const int MAXTIMEOUT = 0x10000000 ;
	    DWORD rv = WAIT_TIMEOUT ;
	    while (_Event < 0 && Millis > 0) {
	       DWORD prd = Millis ;     // set prd = MAX (Millis, MAXTIMEOUT)
	       if (Millis > MAXTIMEOUT) {
	          prd = MAXTIMEOUT ;
	       }
	       rv = ::WaitForSingleObject (_ParkHandle, prd) ;
	       assert (rv == WAIT_OBJECT_0 || rv == WAIT_TIMEOUT, "WaitForSingleObject failed") ;
	       if (rv == WAIT_TIMEOUT) {
	           Millis -= prd ;
	       }
	    }
	    v = _Event ;
	    _Event = 0 ;
	    OrderAccess::fence() ;
	    // If we encounter a nearly simultanous timeout expiry and unpark()
	    // we return OS_OK indicating we awoke via unpark().
	    // Implementor's license -- returning OS_TIMEOUT would be equally valid, however.
	    return (v >= 0) ? OS_OK : OS_TIMEOUT ;
	}
	
```


