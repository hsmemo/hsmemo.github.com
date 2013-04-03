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
void MacroAssembler::call_VM(Register oop_result, address entry_point, int number_of_arguments, bool check_exceptions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「call_VM_base() が生成するコードにより, 指定のエントリポイントの呼び出しを行う.
       (MacroAssembler::call_VM_base(), またはサブクラスでオーバーライドしたメソッドが呼ばれる)」
      ---------------------------------------- -}

	  call_VM_base(oop_result, noreg, noreg, entry_point, number_of_arguments, check_exceptions);
	}
	
```


