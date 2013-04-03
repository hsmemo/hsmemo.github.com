---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) _mutex_lock フィールドに入っている関数を呼び出すだけ.
      ---------------------------------------- -}

	  static int mutex_lock(mutex_t *mx)    { return _mutex_lock(mx); }
	
```


