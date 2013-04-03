---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::throw_pending_exception(JavaThread* thread))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(thread->has_pending_exception(), "must only ne called if there's an exception pending");

  {- -------------------------------------------
  (1) (この関数自体は何もしない. 
       代わりに, この関数を呼び出す MacroAssembler::call_VM() のコード内のチェックで
       例外を検出させて伝搬させるのが目的)
      ---------------------------------------- -}

	  // nothing to do - eventually we should remove this code entirely (see comments @ call sites)
	IRT_END
	
```


