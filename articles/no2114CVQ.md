---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp

### 名前(function name)
```
void Monitor::lock() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Monitor::lock() を呼び出すだけ.
      ---------------------------------------- -}

	  this->lock(Thread::current());
	}
	
```


