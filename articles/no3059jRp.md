---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::call_VM_leaf(Register thread_cache, address entry_point, Register arg_1, Register arg_2, Register arg_3) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数をそれぞれ O1, O2, O3 にコピーした後, MacroAssembler::call_VM_leaf() が生成するコードを実行するだけ」
      ---------------------------------------- -}

	  mov(arg_1, O0);
	  mov(arg_2, O1); assert(arg_2 != O0,                "smashed argument");
	  mov(arg_3, O2); assert(arg_3 != O0 && arg_3 != O1, "smashed argument");
	  call_VM_leaf(thread_cache, entry_point, 3);
	}
	
```


