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
void MacroAssembler::call_VM(Register oop_result, address entry_point, Register arg_1, Register arg_2, bool check_exceptions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数をそれぞれ O1 と O2 にコピーした後, MacroAssembler::call_VM() が生成するコードを実行するだけ」
    
      (なお, O0 はスレッドを入れるために使用されるので O1 から入れる, とのこと)
      ---------------------------------------- -}

	  // O0 is reserved for the thread
	  mov(arg_1, O1);
	  mov(arg_2, O2); assert(arg_2 != O1, "smashed argument");
	  call_VM(oop_result, entry_point, 2, check_exceptions);
	}
	
```


