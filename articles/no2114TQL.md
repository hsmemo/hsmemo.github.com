---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
bool ThreadService::set_thread_monitoring_contention(bool flag) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadService::_thread_monitoring_contention_enabled フィールドを変更する.
      ---------------------------------------- -}

	  MutexLocker m(Management_lock);
	
	  bool prev = _thread_monitoring_contention_enabled;
	  _thread_monitoring_contention_enabled = flag;
	
	  return prev;
	}
	
```


