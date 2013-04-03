---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/windows_x86/vm/thread_windows_x86.hpp

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


