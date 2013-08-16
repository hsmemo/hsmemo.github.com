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
void os::interrupt(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Thread::current() == thread || Threads_lock->owned_by_self(),
	    "possibility of dangling Thread pointer");
	
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
      ---------------------------------------- -}

	  if (!osthread->interrupted()) {
	    osthread->set_interrupted(true);
	    // More than one thread can get here with the same value of osthread,
	    // resulting in multiple notifications.  We do, however, want the store
	    // to interrupted() to be visible to other threads before we execute unpark().
	    OrderAccess::fence();
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

	  // For JSR166. Unpark even if interrupt status already was set
	  if (thread->is_Java_thread())
	    ((JavaThread*)thread)->parker()->unpark();
	
  {- -------------------------------------------
  (1) もし対象のスレッドの _ParkEvent フィールドに 
      ParkEvent オブジェクトが入っていれば (= NULL でなければ), 
      それに対しても os::PlatformEvent::unpark() を呼び出しておく.
      (これは, java.lang.Object.wait() に割り込むための処理.
       See: java.lang.Object.wait())
      ---------------------------------------- -}

	  ParkEvent * ev = thread->_ParkEvent ;
	  if (ev != NULL) ev->unpark() ;
	
	}
	
```


