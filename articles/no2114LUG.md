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
void os::yield() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sched_yield() システムコールを呼び出すだけ.
  
      (<= スレッドが SCHED_OTHER なんだが, この場合も sched_yield() でいいんだっけ?? 要確認 #TODO)
      ---------------------------------------- -}

	  sched_yield();
	}
	
```


