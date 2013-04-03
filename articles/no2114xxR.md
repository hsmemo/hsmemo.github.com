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
OSReturn os::set_native_priority(Thread* thread, int newpri) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) setpriority() システムコールで優先度を変更する.
      
      (ただし, UseThreadPriorities オプションが指定されていない場合や 
       ThreadPriorityPolicy オプションが 0 の場合には, 何もしない)
  
      (なお setpriority() は POSIX の規定では「プロセス単位での制御」だが 
       現在の Linux では「スレッド単位での制御」になっている (See: [here](no3059sSJ.html) for details))
      ---------------------------------------- -}

	  if ( !UseThreadPriorities || ThreadPriorityPolicy == 0 ) return OS_OK;
	
	  int ret = setpriority(PRIO_PROCESS, thread->osthread()->thread_id(), newpri);
	  return (ret == 0) ? OS_OK : OS_ERR;
	}
	
```


