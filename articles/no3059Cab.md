---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_ClassCastException_handler() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) コード生成:
      「オペランドスタック("expression stack")を空にする.」
      ---------------------------------------- -}

	  // expression stack must be empty before entering the VM if an exception
	  // happened
	  __ empty_expression_stack();

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterRuntime::throw_ClassCastException() を呼び出す.」
      ---------------------------------------- -}

	  // load exception object
	  __ call_VM(Oexception,
	             CAST_FROM_FN_PTR(address,
	                              InterpreterRuntime::throw_ClassCastException),
	             Otos_i);

  {- -------------------------------------------
  (1) コード生成:
      「(例外送出なので, ここには決して戻ってこない)」
      ---------------------------------------- -}

	  __ should_not_reach_here();

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


