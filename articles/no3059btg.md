---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/biasedLocking.cpp

### 名前(function name)
```
static void clean_up_cached_monitor_info() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 全 JavaThread 内の cached_monitor_info 情報 (キャッシュしておいた「使用中のロック情報」) を全てクリアするだけ.
      ---------------------------------------- -}

	  // Walk the thread list clearing out the cached monitors
	  for (JavaThread* thr = Threads::first(); thr != NULL; thr = thr->next()) {
	    thr->set_cached_monitor_info(NULL);
	  }
	}
	
```


