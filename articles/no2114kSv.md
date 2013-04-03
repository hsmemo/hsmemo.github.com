---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/linux_x86/vm/thread_linux_x86.hpp

### 名前(function name)
```
  void pd_initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JavaFrameAnchor::clear() で _anchor フィールドを初期化しておく.
      ---------------------------------------- -}

	    _anchor.clear();
	  }
	
```


