---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
VM_GetReceiver::VM_GetReceiver(
    JavaThread* thread, JavaThread* caller_thread, jint depth)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    : VM_GetOrSetLocal(thread, caller_thread, depth, 0) {}
	
```


