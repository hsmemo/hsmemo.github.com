---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) GrowableCache::append() を呼び出すだけ
      ---------------------------------------- -}

	  void append(JvmtiBreakpoint& e)       { _cache.append((GrowableElement *) &e); }
	
```


