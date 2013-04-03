---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.inline.hpp

### 名前(function name)
```
inline bool os::uses_stack_guard_pages() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::win32::is_nt() の結果をリターンするだけ.
      (Windows NT 系列なら true, Windows 9X 系列なら false をリターン)
      ---------------------------------------- -}

	  return os::win32::is_nt();
	}
	
```


