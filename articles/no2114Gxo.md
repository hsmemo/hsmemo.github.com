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
jlong os::thread_cpu_time(Thread* thread, bool user_sys_cpu_time) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下のどちらかの関数を呼び出して, 引数で指定されたスレッドの CPU 使用時間を取得する.
      * NT 系のカーネルの場合:
        GetThreadTimes()
      * そうではない場合:
        timeGetTime()
      ---------------------------------------- -}

	  // This code is copy from clasic VM -> hpi::sysThreadCPUTime
	  // If this function changes, os::is_thread_cpu_time_supported() should too
	  if (os::win32::is_nt()) {
	    FILETIME CreationTime;
	    FILETIME ExitTime;
	    FILETIME KernelTime;
	    FILETIME UserTime;
	
	    if ( GetThreadTimes(thread->osthread()->thread_handle(),
	                    &CreationTime, &ExitTime, &KernelTime, &UserTime) == 0)
	      return -1;
	    else
	      if (user_sys_cpu_time) {
	        return (FT2INT64(UserTime) + FT2INT64(KernelTime)) * 100;
	      } else {
	        return FT2INT64(UserTime) * 100;
	      }
	  } else {
	    return (jlong) timeGetTime() * 1000000;
	  }
	}
	
```


