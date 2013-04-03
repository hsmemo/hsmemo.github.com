---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp

### 名前(function name)
```
void InterpreterMacroAssembler::dispatch_epilog(TosState state, int step) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) InterpreterMacroAssembler::dispatch_next() を呼んでコードを生成するだけ.
      ---------------------------------------- -}

	  dispatch_next(state, step);
	}
	
```


