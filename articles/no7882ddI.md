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
void os::yield_all(int attempts) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sched_yield() を呼び出すだけ.
  
      (<= スレッドが SCHED_OTHER なんだが, この場合も sched_yield() でいいんだっけ?? 要確認 #TODO)
      ---------------------------------------- -}

	  // Yields to all threads, including threads with lower priorities
	  // Threads on Linux are all with same priority. The Solaris style
	  // os::yield_all() with nanosleep(1ms) is not necessary.
	  sched_yield();
	}
	
```


