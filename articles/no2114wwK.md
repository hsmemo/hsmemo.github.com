---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/linux_sparc/vm/os_linux_sparc.cpp

### 名前(function name)
```
size_t os::Linux::default_guard_size(os::ThreadType thr_type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaThread の場合には 0, そうでない場合には 1 ページ分とする.
      (JavaThread の場合には HotSpot が独自のガードページを張るので, glibc のガードページは不要)
      ---------------------------------------- -}

	  // Creating guard page is very expensive. Java thread has HotSpot
	  // guard page, only enable glibc guard page for non-Java threads.
	  return (thr_type == java_thread ? 0 : page_size());
	}
	
```


