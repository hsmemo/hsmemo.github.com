---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) os::NakedYield() を呼び出すだけ.
      ---------------------------------------- -}

	void os::yield() {  os::NakedYield(); }
	
```


