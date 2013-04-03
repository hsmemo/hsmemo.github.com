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
void SurrogateLockerThread::manipulatePLL(SLT_msg_type msg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は _monitor フィールド (コンストラクタ引数で渡された Monitor オブジェクト) のロックを取った状態で行う)
      ---------------------------------------- -}

	  MutexLockerEx x(&_monitor, Mutex::_no_safepoint_check_flag);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_buffer == empty, "Should be empty");
	  assert(msg != empty, "empty message");

  {- -------------------------------------------
  (1) _buffer フィールドに msg 引数のメッセージをセットする.
      ---------------------------------------- -}

	  _buffer = msg;

  {- -------------------------------------------
  (1) _monitor フィールドに対して Monitor::notify() を呼んで SurrogateLockerThread スレッドを起床させる.
      その後, _monitor フィールドに対して Monitor::wait() して, SurrogateLockerThread の処理が終わるまで待機する.
      (この処理は, _buffer フィールドが空になっていなければ, 空になるまで何度でも繰り返す)
      ---------------------------------------- -}

	  while (_buffer != empty) {
	    _monitor.notify();
	    _monitor.wait(Mutex::_no_safepoint_check_flag);
	  }
	}
	
```


