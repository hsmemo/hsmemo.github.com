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
void MacroAssembler::call_VM(Register oop_result, Register last_java_sp, address entry_point, Register arg_1, Register arg_2, Register arg_3, bool check_exceptions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数をそれぞれ O1, O2, O3 にコピーした後, MacroAssembler::call_VM() が生成するコードを実行するだけ.
       なお, この場合は last_java_sp の値を指定して呼び出す.」
    
      (なお, O0 はスレッドを入れるために使用されるので O1 から入れる, とのこと)
      ---------------------------------------- -}

	  // O0 is reserved for the thread
	  mov(arg_1, O1);
	  mov(arg_2, O2); assert(arg_2 != O1,                "smashed argument");
	  mov(arg_3, O3); assert(arg_3 != O1 && arg_3 != O2, "smashed argument");
	  call_VM(oop_result, last_java_sp, entry_point, 3, check_exceptions);
	}
	
```


