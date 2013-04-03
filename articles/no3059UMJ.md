---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/javaCalls.cpp

### 名前(function name)
```
void JavaCalls::call(JavaValue* result, methodHandle method, JavaCallArguments* args, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Check if we need to wrap a potential OS exception handler around thread
	  // This is used for e.g. Win32 structured exception handlers
	  assert(THREAD->is_Java_thread(), "only JavaThreads can make JavaCalls");

  {- -------------------------------------------
  (1) os::os_exception_wrapper() 経由で, JavaCalls::call_helper() を呼び出す.
  
      (なお, os::os_exception_wrapper() を経由するのは, 
       この時点までに積まれたスタック内のネイティブコードが
       どこかで勝手に独自の例外ハンドラをインストールしているかもしれないため)
      ---------------------------------------- -}

	  // Need to wrap each and everytime, since there might be native code down the
	  // stack that has installed its own exception handlers
	  os::os_exception_wrapper(call_helper, result, &method, args, THREAD);
	}
	
```


