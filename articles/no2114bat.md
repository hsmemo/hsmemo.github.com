---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp

### 名前(function name)
```
void os::PlatformEvent::unpark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int v, AnyWaiters ;

  {- -------------------------------------------
  (1) _Event の値が 0 より大きければ (= つまり 1 であれば), ここでリターンする.
      (なお, このプロセッサの store buffer 内の値を見てしまう恐れがあるので, 
       OrderAccess::fence() を張って確認している)
    
      そうでなければ Atomic::cmpxchg() で _Event の値を 1 増加させておく.
      (Atomic::cmpxchg() が失敗した場合は, 成功するまで以上の処理を繰り返す)
      ---------------------------------------- -}

	  for (;;) {
	      v = _Event ;
	      if (v > 0) {
	         // The LD of _Event could have reordered or be satisfied
	         // by a read-aside from this processor's write buffer.
	         // To avoid problems execute a barrier and then
	         // ratify the value.
	         OrderAccess::fence() ;
	         if (_Event == v) return ;
	         continue ;
	      }
	      if (Atomic::cmpxchg (v+1, &_Event, v) == v) break ;
	  }

  {- -------------------------------------------
  (1) 変更前の _Event の値が負値だった場合は
      park() で待機しているスレッドがいる(かもしれない)ので, 
      pthread_cond_signal() で起こしてやる.
    
      (ただし, _nParked フィールドが 0 の場合は起床処理は行わない.
       本当に寝ているスレッドがいる場合には _nParked フィールドが 1 以上になっているはずなので.
       See: os::PlatformEvent::park()
       
       ついでに, この _nParked フィールドの読み取りは 
       _mutex で保護された critical section 内で行う)
  
      (また, WorkAroundNPTLTimedWaitHang オプションが指定されている場合は, 
       中途半端な cond_var を見てしまわないよう, 
       pthread_cond_signal() の呼び出しを critical section 内で行う.
       See: WorkAroundNPTLTimedWaitHang)
      ---------------------------------------- -}

	  if (v < 0) {
	     // Wait for the thread associated with the event to vacate
	     int status = pthread_mutex_lock(_mutex);
	     assert_status(status == 0, status, "mutex_lock");
	     AnyWaiters = _nParked ;
	     assert (AnyWaiters == 0 || AnyWaiters == 1, "invariant") ;
	     if (AnyWaiters != 0 && WorkAroundNPTLTimedWaitHang) {
	        AnyWaiters = 0 ;
	        pthread_cond_signal (_cond);
	     }
	     status = pthread_mutex_unlock(_mutex);
	     assert_status(status == 0, status, "mutex_unlock");
	     if (AnyWaiters != 0) {
	        status = pthread_cond_signal(_cond);
	        assert_status(status == 0, status, "cond_signal");
	     }
	  }
	
  {- -------------------------------------------
  (1) (なお, critical section を抜けてから pthread_cond_signal() を出しているが, 
       これによりよくある種類の futile wakeup が避けられる.                         (<= どういう futile wakeup だろう?? #TODO)
       逆に, 非常にまれなケースで spurious wakeup を引き起こしてしまうこともあるが, 
       起こされた側が単に park し直すだけなので問題ない.)
      ---------------------------------------- -}

	  // Note that we signal() _after dropping the lock for "immortal" Events.
	  // This is safe and avoids a common class of  futile wakeups.  In rare
	  // circumstances this can cause a thread to return prematurely from
	  // cond_{timed}wait() but the spurious wakeup is benign and the victim will
	  // simply re-test the condition and re-park itself.
	}
	
```


