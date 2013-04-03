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
bool os::unguard_memory(char* addr, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) linux_mprotect() を呼び出すだけ.
      ---------------------------------------- -}

	  return linux_mprotect(addr, size, PROT_READ|PROT_WRITE);
	}
	
```


