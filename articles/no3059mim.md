---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp
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
	  __ push(c_rarg0);
	  __ push(c_rarg1);
	  __ push(c_rarg2);
	  __ push(c_rarg3);
	  __ mov(c_rarg2, rax);  // Pass itos
	#ifdef _WIN64
	  __ movflt(xmm3, xmm0); // Pass ftos
	#endif
	  __ call_VM(noreg,
	             CAST_FROM_FN_PTR(address, SharedRuntime::trace_bytecode),
	             c_rarg1, c_rarg2, c_rarg3);
	  __ pop(c_rarg3);
	  __ pop(c_rarg2);
	  __ pop(c_rarg1);
	  __ pop(c_rarg0);
	  __ pop(state);

  {- -------------------------------------------
  (1) コード生成:
      「リターン」
      ---------------------------------------- -}

	  __ ret(0);                                   // return from result handler
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


