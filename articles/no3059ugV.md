---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)
// This does not do anything on Solaris. This is basically a hook for being
// able to use structured exception handling (thread-local exception filters) on, e.g., Win32.


### 名前(function name)
```
void os::os_exception_wrapper(java_call_t f, JavaValue* value, methodHandle* method, JavaCallArguments* args, Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 単に引数で渡された関数を呼び出すだけ.
      ---------------------------------------- -}

	  f(value, method, args, thread);
	}
	
```


