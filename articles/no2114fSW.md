---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)

```
// hrtime_t gethrvtime() return value includes
// user time but does not include system time
```

### 名前(function name)
```
jlong os::current_thread_cpu_time() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) gethrvtime() を呼び出すだけ.
      ---------------------------------------- -}

	  return (jlong) gethrvtime();
	}
	
```


