---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)

```
// Non-product code
#ifndef PRODUCT
```

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_trace_code(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「SharedRuntime::trace_bytecode() を呼び出す.
       (なお, 呼び出しの前後で O7 と state の退避復帰も行っている)」
      ---------------------------------------- -}

	  __ push(state);
	  __ mov(O7, Lscratch); // protect return address within interpreter
	
	  // Pass a 0 (not used in sparc) and the top of stack to the bytecode tracer
	  __ mov( Otos_l2, G3_scratch );
	  __ call_VM(noreg, CAST_FROM_FN_PTR(address, SharedRuntime::trace_bytecode), G0, Otos_l1, G3_scratch);
	  __ mov(Lscratch, O7); // restore return address
	  __ pop(state);

  {- -------------------------------------------
  (1) コード生成:
      「リターン」
      ---------------------------------------- -}

	  __ retl();
	  __ delayed()->nop();
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


