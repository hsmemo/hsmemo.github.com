---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp

### 名前(function name)
```
void SurrogateLockerThread::loop() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  BasicLock pll_basic_lock;
	  SLT_msg_type msg;
	  debug_only(unsigned int owned = 0;)
	
  {- -------------------------------------------
  (1) (以下の while ループが SurrogateLockerThread スレッドのメインループ)
      ---------------------------------------- -}

	  while (/* !isTerminated() */ 1) {

    {- -------------------------------------------
  (1.1) _buffer フィールドにメッセージが到着するまで, 
        _monitor フィールド (コンストラクタ引数で渡された Monitor オブジェクト) に対して
        Monitor::wait() を呼んで待機する
  
        メッセージが到着したら, それを msg 局所変数にコピーして以下のブロックを抜ける.
        ---------------------------------------- -}

	    {
	      MutexLocker x(&_monitor);
	      // Since we are a JavaThread, we can't be here at a safepoint.
	      assert(!SafepointSynchronize::is_at_safepoint(),
	             "SLT is a JavaThread");
	      // wait for msg buffer to become non-empty
	      while (_buffer == empty) {
	        _monitor.notify();
	        _monitor.wait();
	      }
	      msg = _buffer;
	    }

    {- -------------------------------------------
  (1.1) 届いたメッセージに応じて, 処理は３通りに分かれる.
  
        * acquirePLL の場合:
          instanceRefKlass::acquire_pending_list_lock() を呼び出して, 
          java.lang.ref.Reference のロックを取得する.
  
        * releaseAndNotifyPLL の場合:
          instanceRefKlass::release_and_notify_pending_list_lock() を呼び出して, 
          java.lang.ref.Reference のロックを開放する.
  
        * empty の場合:
          何もしない
        ---------------------------------------- -}

	    switch(msg) {
	      case acquirePLL: {
	        instanceRefKlass::acquire_pending_list_lock(&pll_basic_lock);
	        debug_only(owned++;)
	        break;
	      }
	      case releaseAndNotifyPLL: {
	        assert(owned > 0, "Don't have PLL");
	        instanceRefKlass::release_and_notify_pending_list_lock(&pll_basic_lock);
	        debug_only(owned--;)
	        break;
	      }
	      case empty:
	      default: {
	        guarantee(false,"Unexpected message in _buffer");
	        break;
	      }
	    }

    {- -------------------------------------------
  (1.1) _buffer フィールドを空の状態に戻す.
        ついでに, _monitor フィールドに対して Monitor::notify() を呼んで, 処理の終了を通知しておく.
        ---------------------------------------- -}

	    {
	      MutexLocker x(&_monitor);
	      // Since we are a JavaThread, we can't be here at a safepoint.
	      assert(!SafepointSynchronize::is_at_safepoint(),
	             "SLT is a JavaThread");
	      _buffer = empty;
	      _monitor.notify();
	    }
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!_monitor.owned_by_self(), "Should unlock before exit.");
	}
	
```


