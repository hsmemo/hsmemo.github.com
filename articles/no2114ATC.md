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
void os::interrupt(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Thread::current() == thread || Threads_lock->owned_by_self(), "possibility of dangling Thread pointer");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread* osthread = thread->osthread();
	
  {- -------------------------------------------
  (1) 対象スレッドの _SleepEvent フィールドにある ParkEvent オブジェクトに対して
      os::PlatformEvent::unpark() を呼び出しておく.
      (これは java.lang.Thread.sleep() に割り込むための処理. 
       See: java.lang.Thread.sleep())
  
      ただし, 既に対象のスレッドが割り込まれていた場合には, この処理は行わない.
      この確認処理も含めて, 処理は以下の順で行う.
      (1) OSThread::interrupted() で, 処理対象のスレッドの interrupted フラグを確認.
      (2) OSThread::set_interrupted() で, 処理対象のスレッドの interrupted フラグを立てる.
      (3) OrderAccess::fence() でメモリバリアを張り, 
          この後の load/store よりも先に interrupted フラグの変更が visible になるようにしておく.
      (4) os::PlatformEvent::unpark() を呼び出す.
  
      なお, os::sleep() の待機処理は, _SleepEvent に対する park() ではなく, poll() になっていることもある.
      ただし, poll() のケースについては後の thr_kill() で割り込むので問題ない.
      ---------------------------------------- -}

	  int isInterrupted = osthread->interrupted();
	  if (!isInterrupted) {
	      osthread->set_interrupted(true);
	      OrderAccess::fence();
	      // os::sleep() is implemented with either poll (NULL,0,timeout) or
	      // by parking on _SleepEvent.  If the former, thr_kill will unwedge
	      // the sleeper by SIGINTR, otherwise the unpark() will wake the sleeper.
	      ParkEvent * const slp = thread->_SleepEvent ;
	      if (slp != NULL) slp->unpark() ;
	  }
	
  {- -------------------------------------------
  (1) もし対象のスレッドが JavaThread であれば, 
      JavaThread::parker() で Parker オブジェクトを取得して
      Parker::unpark() を呼び出しておく.
      (これは, java.util.concurrent.locks.LockSupport.park() に割り込むための処理.
       See: java.util.concurrent.locks.LockSupport.park())
      ---------------------------------------- -}

	  // For JSR166:  unpark after setting status but before thr_kill -dl
	  if (thread->is_Java_thread()) {
	    ((JavaThread*)thread)->parker()->unpark();
	  }
	
  {- -------------------------------------------
  (1) もし対象のスレッドの _ParkEvent フィールドに 
      ParkEvent オブジェクトが入っていれば (= NULL でなければ), 
      それに対しても os::PlatformEvent::unpark() を呼び出しておく.
      (これは, java.lang.Object.wait() に割り込むための処理.
       See: java.lang.Object.wait())
      ---------------------------------------- -}

	  // Handle interruptible wait() ...
	  ParkEvent * const ev = thread->_ParkEvent ;
	  if (ev != NULL) ev->unpark() ;
	
  {- -------------------------------------------
  (1) 上で調べた際に対象スレッドが割り込まれていなかった場合には, 
      thr_kill() でシグナルも送っておく.
      (これは os::sleep() に割り込むための処理. 上述)
    
      (ついでに(プロファイル情報の記録)も行っている.
       See: RuntimeService)
      ---------------------------------------- -}

	  // When events are used everywhere for os::sleep, then this thr_kill
	  // will only be needed if UseVMInterruptibleIO is true.
	
	  if (!isInterrupted) {
	    int status = thr_kill(osthread->thread_id(), os::Solaris::SIGinterrupt());
	    assert_status(status == 0, status, "thr_kill");
	
	    // Bump thread interruption counter
	    RuntimeService::record_thread_interrupt_signaled_count();
	  }
	}
	
```


