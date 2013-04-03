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
  (1) _cond_wait フィールドに入っている関数を呼び出すだけ.
      ---------------------------------------- -}

	  static int cond_wait(cond_t *cv, mutex_t *mx) { return _cond_wait(cv, mx); }
	
```


