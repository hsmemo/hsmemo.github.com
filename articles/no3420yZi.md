---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/threadCritical_solaris.cpp

### 名前(function name)
```
void ThreadCritical::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) initialized 大域変数を true にセットするだけ.
      ---------------------------------------- -}

	  // This method is called at the end of os::init(). Until
	  // then, we don't do real locking.
	  initialized = true;
	}
	
```


