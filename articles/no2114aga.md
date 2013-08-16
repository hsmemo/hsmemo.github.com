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
void os::interrupt(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!thread->is_Java_thread() || Thread::current() == thread || Threads_lock->owned_by_self(),
	         "possibility of dangling Thread pointer");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread* osthread = thread->osthread();

  {- -------------------------------------------
  (1) OSThread::set_interrupted() を呼んで, 
      処理対象のスレッドの interrupted フラグを立てておく.
    
      (ついでに, OrderAccess::release() でメモリバリアも張って, 
       この後の store よりも先に interrupted フラグの変更が visible になるようにしておく)
      ---------------------------------------- -}

	  osthread->set_interrupted(true);
	  // More than one thread can get here with the same value of osthread,
	  // resulting in multiple notifications.  We do, however, want the store
	  // to interrupted() to be visible to other threads before we post
	  // the interrupt event.
	  OrderAccess::release();

  {- -------------------------------------------
  (1) 対象のスレッドが保持しているイベントオブジェクトに対し, SetEvent() でシグナルを送る.
      (これは java.lang.Thread.sleep() に割り込むための処理. 
       See: java.lang.Thread.sleep())
      ---------------------------------------- -}

	  SetEvent(osthread->interrupt_event());

  {- -------------------------------------------
  (1) もし対象のスレッドが JavaThread であれば, 
      JavaThread::parker() で Parker オブジェクトを取得して
      Parker::unpark() を呼び出しておく.
      (これは, java.util.concurrent.locks.LockSupport.park() に割り込むための処理.
       See: java.util.concurrent.locks.LockSupport.park())
      ---------------------------------------- -}

	  // For JSR166:  unpark after setting status
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


