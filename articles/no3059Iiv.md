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
address os::current_stack_base() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) current_stack_region() を呼んで, スタックの底にあたるアドレスとスタックサイズを取得し, 
      その2つを足した値をリターン.
      ---------------------------------------- -}

	  address bottom;
	  size_t size;
	  current_stack_region(&bottom, &size);
	  return (bottom + size);
	}
	
```


