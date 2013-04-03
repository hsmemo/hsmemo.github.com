---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp
### 説明(description)
void MacroAssembler::call_VM_leaf(address entry_point, int number_of_arguments) {



### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「call_VM_leaf_base() が生成するコードにより, 引数で指定されたエントリポイントを呼び出す.
       (MacroAssembler::call_VM_leaf_base(), またはサブクラスでオーバーライドしたメソッドが呼ばれる)」
      ---------------------------------------- -}

	  call_VM_leaf_base(entry_point, number_of_arguments);
	}
	
```


