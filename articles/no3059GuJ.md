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
void os::pd_start_thread(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread * osthread = thread->osthread();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(osthread->get_state() != INITIALIZED, "just checking");

  {- -------------------------------------------
  (1) 生成したスレッドの startThread_lock に対して Monitor::notify() を呼び出し, 初期化が完了したことを伝える.
      (See: java_start())
      ---------------------------------------- -}

	  Monitor* sync_with_child = osthread->startThread_lock();
	  MutexLockerEx ml(sync_with_child, Mutex::_no_safepoint_check_flag);
	  sync_with_child->notify();
	}
	
```


