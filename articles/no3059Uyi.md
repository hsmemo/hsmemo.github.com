---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.hpp
### 説明(description)

```
  // NPTL is always floating stack. LinuxThreads could be using floating
  // stack or fixed stack.
```


### 本体部(body)
```
  {- -------------------------------------------
  (1) (単なる getter method)
      ---------------------------------------- -}

	  static bool is_floating_stack()             { return _is_floating_stack; }
	
```


