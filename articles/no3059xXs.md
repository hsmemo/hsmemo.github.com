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
bool os::unguard_memory(char* addr, size_t bytes) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VirtualProtect() を呼び出すだけ.
      ---------------------------------------- -}

	  DWORD old_status;
	  return VirtualProtect(addr, bytes, PAGE_READWRITE, &old_status) != 0;
	}
	
```


