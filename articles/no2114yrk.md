---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.hpp

### 名前(function name)
```
  void set_pending_async_exception(oop e) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    _pending_async_exception = e;
	    _special_runtime_exit_condition = _async_exception;
	    set_has_async_exception();
	  }
	
```


