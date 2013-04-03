---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vmThread.cpp

### 名前(function name)
```
void VMThread::destroy() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VMThread 用のメモリ領域を開放する.
      ---------------------------------------- -}

	  if (_vm_thread != NULL) {
	    delete _vm_thread;
	    _vm_thread = NULL;      // VM thread is gone
	  }
	}
	
```


