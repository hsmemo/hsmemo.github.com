---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_safept_entry_for(
        TosState state,
        address runtime_entry) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM() が生成するコードによって,
        引数で指定されたアドレス(runtime_entry)を呼び出す.
        その後, normal_table から次の命令を dispatch する.」
      ---------------------------------------- -}

	  __ push(state);
	  __ call_VM(noreg, runtime_entry);
	  __ dispatch_via(vtos, Interpreter::_normal_table.table_for(vtos));

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


