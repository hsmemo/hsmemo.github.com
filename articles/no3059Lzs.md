---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/windows_x86/vm/threadLS_windows_x86.cpp

### 名前(function name)
```
void ThreadLocalStorage::pd_set_thread(Thread* thread)  {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::thread_local_storage_at_put() で TLS に値をセットする.
      ---------------------------------------- -}

	  os::thread_local_storage_at_put(ThreadLocalStorage::thread_index(), thread);
	}
	
```


