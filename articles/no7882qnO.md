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
void os::yield_all(int attempts) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Sleep() を呼び出すだけ.
      ---------------------------------------- -}

	  // Yields to all threads, including threads with lower priorities
	  Sleep(1);
	}
	
```


