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
bool os::create_stack_guard_pages(char* addr, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::commit_memory() を呼んで, 指定範囲のメモリ領域をコミットする.
      ---------------------------------------- -}

	  return os::commit_memory(addr, size);
	}
	
```


