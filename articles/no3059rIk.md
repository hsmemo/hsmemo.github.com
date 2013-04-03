---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/exceptions.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) Exceptions::_throw_msg() に変換されるだけ.
      ---------------------------------------- -}

	#define THROW_MSG(name, message)                    \
	  { Exceptions::_throw_msg(THREAD_AND_LOCATION, name, message); return;  }
	
```


