---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.hpp
### 説明(description)

```
  // Whenever a thread transitions from native to vm/java it must suspend
  // if external|deopt suspend is present.
```

### 名前(function name)
```
  bool is_suspend_after_native() const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _suspend_flags 中の _external_suspend ビット, もしくは _deopt_suspend ビットが立っていれば, true をリターンする.
      (See: [here](no2114zBI.html) for details)
      ---------------------------------------- -}

	    return (_suspend_flags & (_external_suspend | _deopt_suspend) ) != 0;
	  }
	
```


