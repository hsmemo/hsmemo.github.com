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
void MacroAssembler::call_VM(Register oop_result, address entry_point, Register arg_1, bool check_exceptions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数を O1 にコピーした後, MacroAssembler::call_VM() が生成するコードを実行するだけ」
    
      (なお, O0 はスレッドを入れるために使用されるので O1 から入れる, とのこと)
      ---------------------------------------- -}

	  // O0 is reserved for the thread
	  mov(arg_1, O1);
	  call_VM(oop_result, entry_point, 1, check_exceptions);
	}
	
```


