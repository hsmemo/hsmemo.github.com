---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentMark.cpp

### 名前(function name)
```
void ConcurrentMark::set_non_marking_state() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // We set the global marking state to some default values when we're
	  // not doing marking.
	  clear_marking_state();
	  _active_tasks = 0;
	  clear_concurrent_marking_in_progress();
	}
	
```


