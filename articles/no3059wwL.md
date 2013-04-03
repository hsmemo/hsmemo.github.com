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
void MacroAssembler::call_VM_leaf(address entry_point, Register arg_0, Register arg_1, Register arg_2) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「引数を積んだ後, MacroAssembler::call_VM_leaf() が生成するコードを実行するだけ.」
      ---------------------------------------- -}

	  LP64_ONLY(assert(arg_0 != c_rarg2, "smashed arg"));
	  LP64_ONLY(assert(arg_1 != c_rarg2, "smashed arg"));
	  pass_arg2(this, arg_2);
	  LP64_ONLY(assert(arg_0 != c_rarg1, "smashed arg"));
	  pass_arg1(this, arg_1);
	  pass_arg0(this, arg_0);
	  call_VM_leaf(entry_point, 3);
	}
	
```


