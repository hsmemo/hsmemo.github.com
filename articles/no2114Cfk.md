---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/classLoadingService.cpp
### 説明(description)

```
// Caller to this function must own Management_lock
```

### 名前(function name)
```
void ClassLoadingService::reset_trace_class_unloading() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CommandLineFlags::boolAtPut() を使って
      TraceClassUnloading オプションの値を変更する.
    
      値は, MemoryService と ClassLoadingService のどちらか片方でも verbose 状態になっていれば true, 
      どちらも verbose 状態でなければ false.
      ---------------------------------------- -}

	  assert(Management_lock->owned_by_self(), "Must own the Management_lock");
	  bool value = MemoryService::get_verbose() || ClassLoadingService::get_verbose();
	  bool succeed = CommandLineFlags::boolAtPut((char*)"TraceClassUnloading", &value, MANAGEMENT);
	  assert(succeed, "Setting TraceClassUnLoading flag fails");
	}
	
```


