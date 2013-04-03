---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/linux_x86/vm/os_linux_x86.cpp

### 名前(function name)
```
size_t os::current_stack_size() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) current_stack_region() を呼んでスタックサイズを取得し, それをリターン.
      ---------------------------------------- -}

	  // stack size includes normal stack and HotSpot guard pages
	  address bottom;
	  size_t size;
	  current_stack_region(&bottom, &size);
	  return size;
	}
	
```


