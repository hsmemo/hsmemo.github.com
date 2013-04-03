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
void os::pd_start_thread(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ResumeThread() システムコールで, 生成したスレッドを実際に開始させる.
  
      (スレッドを CREATE_SUSPENDED で生成しているため, 生成直後には開始されず, この時点から実行が始まる.
       os::create_thread())
      ---------------------------------------- -}

	  DWORD ret = ResumeThread(thread->osthread()->thread_handle());

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Returns previous suspend state:
	  // 0:  Thread was not suspended
	  // 1:  Thread is running now
	  // >1: Thread is still suspended.
	  assert(ret != SYS_THREAD_ERROR, "StartThread failed"); // should propagate back
	}
	
```


