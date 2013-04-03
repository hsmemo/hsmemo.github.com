---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
JVM_ENTRY_NO_ENV(void, JVM_Halt(jint code))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) before_exit() を呼び出す.
      ---------------------------------------- -}

	  before_exit(thread);

  {- -------------------------------------------
  (1) vm_exit() を呼び出す.
      ---------------------------------------- -}

	  vm_exit(code);
	
```


