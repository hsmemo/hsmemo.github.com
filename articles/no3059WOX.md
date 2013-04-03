---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp

### 名前(function name)
```
void MacroAssembler::call_VM(Register oop_result,
                             Register last_java_sp,
                             address entry_point,
                             Register arg_1,
                             Register arg_2,
                             bool check_exceptions) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) コード生成:
      「引数を積んだ後, MacroAssembler::call_VM() が生成するコードを実行するだけ.
       なお, この場合は last_java_sp の値を指定して呼び出す.」
      ---------------------------------------- -}

	  LP64_ONLY(assert(arg_1 != c_rarg2, "smashed arg"));
	  pass_arg2(this, arg_2);
	  pass_arg1(this, arg_1);
	  call_VM(oop_result, last_java_sp, entry_point, 2, check_exceptions);
	}
	
```


