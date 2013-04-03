---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.hpp

### 名前(function name)
```
  inline static bool do_call_back() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _state が _not_synchronized でなければ true をリターン.
      ---------------------------------------- -}

	    return (_state != _not_synchronized);
	  }
	
```


