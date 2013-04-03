---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
bool os::unguard_memory(char* addr, size_t bytes) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) solaris_mprotect() を呼び出すだけ.
      ---------------------------------------- -}

	  return solaris_mprotect(addr, bytes, PROT_READ|PROT_WRITE);
	}
	
```


