---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreter_x86_64.cpp
### 説明(description)

```
// Abstract method entry
// Attempt to execute abstract method. Throw exception
```

### 名前(function name)
```
address InterpreterGenerator::generate_abstract_entry(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (生成したコードが実行される時点では, レジスタには以下の値が入っているはず)
      ---------------------------------------- -}

	  // rbx: methodOop
	  // r13: sender SP
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry_point = __ pc();
	
	  // abstract method entry
	
  {- -------------------------------------------
  (1) コード生成:
     「オペランドスタック("expression stack")を空にし, 
       r13(bcp)や r14(locals) を元に戻しておく」
  
      (きちんと例外ハンドリング処理ができるよう, 
       インタープリタ用のレジスタを正常な状態に戻しておく処理??)
      ---------------------------------------- -}

	  //  pop return address, reset last_sp to NULL
	  __ empty_expression_stack();
	  __ restore_bcp();      // rsi must be correct for exception handler   (was destroyed)
	  __ restore_locals();   // make sure locals pointer is correct as well (was destroyed)
	
  {- -------------------------------------------
  (1) コード生成:
      「AbstractMethodError を送出するだけ」
      ---------------------------------------- -}

	  // throw exception
	  __ call_VM(noreg, CAST_FROM_FN_PTR(address,
	                             InterpreterRuntime::throw_AbstractMethodError));
	  // the call_VM checks for exception, so we should never return here.
	  __ should_not_reach_here();
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry_point;
	}
	
```


