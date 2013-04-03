---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/fprofiler.cpp

### 名前(function name)
```
void FlatProfiler::record_vm_operation() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (Universe::heap()->is_gc_active()) {
	    FlatProfiler::received_gc_ticks += 1;
	    return;
	  }
	
	  if (DeoptimizationMarker::is_active()) {
	    FlatProfiler::deopt_ticks += 1;
	    return;
	  }
	
	  FlatProfiler::vm_operation_ticks += 1;
	}
	
```


