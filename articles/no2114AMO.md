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
  bool is_external_suspend() const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    return (_suspend_flags & _external_suspend) != 0;
	  }
	
```


