---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)
// guard_memory and unguard_memory only happens within stack guard pages.
// Since ISM pertains only to the heap, guard and unguard memory should not
/// happen with an ISM region.


### 名前(function name)
```
bool os::guard_memory(char* addr, size_t bytes) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) solaris_mprotect() を呼び出すだけ.
      ---------------------------------------- -}

	  return solaris_mprotect(addr, bytes, PROT_NONE);
	}
	
```


