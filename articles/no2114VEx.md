---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp

### 名前(function name)
```
static jint get_vm_thread_count() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VmThreadCountClosure を使ってスレッド数を数え, その結果をリターンする.
      ---------------------------------------- -}

	  VmThreadCountClosure vmtcc;
	  {
	    MutexLockerEx ml(Threads_lock);
	    Threads::threads_do(&vmtcc);
	  }
	
	  return vmtcc.count();
	}
	
```


