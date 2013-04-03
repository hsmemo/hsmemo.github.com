---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp

### 名前(function name)
```
  void do_monitor(ObjectMonitor* mid) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 処理対象の ObjectMonitor が, カレントスレッドがロックを握っているものであれば, 
      ObjectMonitor::complete_exit() を呼び出してロックの開放処理を行う.
      ---------------------------------------- -}

	    if (mid->owner() == THREAD) {
	      (void)mid->complete_exit(CHECK);
	    }
	  }
	
```


