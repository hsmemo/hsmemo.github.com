---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRawMonitor.cpp

### 名前(function name)
```
JvmtiRawMonitor::~JvmtiRawMonitor() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	#ifdef ASSERT
	  FreeHeap(_name);
	#endif
	  _magic = 0;
	}
	
```


