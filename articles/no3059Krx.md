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
inline bool os::allocate_stack_guard_pages() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) true を返すだけ.
      ---------------------------------------- -}

	  assert(uses_stack_guard_pages(), "sanity check");
	  return true;
	}
	
```


