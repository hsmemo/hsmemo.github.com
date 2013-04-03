---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/threadCritical_windows.cpp

### 名前(function name)
```
void ThreadCritical::release() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(lock_owner == -1, "Mutex being deleted while owned.");
	  assert(lock_count == -1, "Mutex being deleted while recursively locked");
	  assert(lock_event != NULL, "Sanity check");

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  CloseHandle(lock_event);
	}
	
```


