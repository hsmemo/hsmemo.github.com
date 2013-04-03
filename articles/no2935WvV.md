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
IRT_ENTRY(void, InterpreterRuntime::_breakpoint(JavaThread* thread, methodOopDesc* method, address bcp))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiExport::post_raw_breakpoint() を呼び出すだけ.
      ---------------------------------------- -}

	  JvmtiExport::post_raw_breakpoint(thread, method, bcp);
	IRT_END
	
```


