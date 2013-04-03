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
address TemplateInterpreterGenerator::generate_ArrayIndexOutOfBounds_handler(const char* name) {
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

	  // expression stack must be empty before entering the VM if an exception happened
	  __ empty_expression_stack();

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterRuntime::throw_ArrayIndexOutOfBoundsException() を呼び出す.」
      ---------------------------------------- -}

	  // convention: expect aberrant index in register G3_scratch, then shuffle the
	  // index to G4_scratch for the VM call
	  __ mov(G3_scratch, G4_scratch);
	  __ set((intptr_t)name, G3_scratch);
	  __ call_VM(Oexception, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_ArrayIndexOutOfBoundsException), G3_scratch, G4_scratch);

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


