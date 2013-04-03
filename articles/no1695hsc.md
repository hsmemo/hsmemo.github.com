---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreter_x86_64.cpp

### 名前(function name)
```
address AbstractInterpreterGenerator::generate_slow_signature_handler() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) (この時点では, 各レジスタには以下の値が入っているはず)
      ---------------------------------------- -}

	  // rbx: method
	  // r14: pointer to locals
	  // c_rarg3: first stack arg - wordSize

  {- -------------------------------------------
  (1) コード生成:
      「引数をセットする.」
      ---------------------------------------- -}

	  __ mov(c_rarg3, rsp);

  {- -------------------------------------------
  (1) コード生成:
      「(以降の引数のセット処理のために) SP をずらしておく.」
      ---------------------------------------- -}

	  // adjust rsp
	  __ subptr(rsp, 14 * wordSize);

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterRuntime::slow_signature_handler() を呼び出す.」
  
      なお, 呼び出し時の引数は以下の通り.
      * 第1引数(O0): r15_thread レジスタの値 (x86_64 なので. See: MacroAssembler::call_VM())
      * 第2引数(O1): rbx の値 ("method")
      * 第3引数(O2): r14 の値 ("pointer to locals")
      * 第4引数(O3): c_rarg3 の値 ("first stack arg - wordSize")
      ---------------------------------------- -}

	  __ call_VM(noreg,
	             CAST_FROM_FN_PTR(address,
	                              InterpreterRuntime::slow_signature_handler),
	             rbx, r14, c_rarg3);
	
  {- -------------------------------------------
  (1) (この時点では, 各レジスタには以下の値が入っているはず)
      ---------------------------------------- -}

	  // rax: result handler
	
  {- -------------------------------------------
  (1) (この時点では, スタックフレームには以下の値が入っているはず)
      (この段階で, InterpreterRuntime::slow_signature_handler() によって
       引数は全てスタック上にコピーされている.
       とはいえ, スタックではなくレジスタで渡す引数もあるので, 
       以下の処理でレジスタで渡すものについてはレジスタに移しておく.)
      ---------------------------------------- -}

	  // Stack layout:
	  // rsp: 5 integer args (if static first is unused)
	  //      1 float/double identifiers
	  //      8 double args
	  //        return address
	  //        stack args
	  //        garbage
	  //        expression stack bottom
	  //        bcp (NULL)
	  //        ...
	
  {- -------------------------------------------
  (1) コード生成:
      「まず浮動小数点引数について, レジスタで渡す物を XMM レジスタに移す.」
      ---------------------------------------- -}

	  // Do FP first so we can use c_rarg3 as temp
	  __ movl(c_rarg3, Address(rsp, 5 * wordSize)); // float/double identifiers
	
	  for (int i = 0; i < Argument::n_float_register_parameters_c; i++) {
	    const XMMRegister r = as_XMMRegister(i);
	
	    Label d, done;
	
	    __ testl(c_rarg3, 1 << i);
	    __ jcc(Assembler::notZero, d);
	    __ movflt(r, Address(rsp, (6 + i) * wordSize));
	    __ jmp(done);
	    __ bind(d);
	    __ movdbl(r, Address(rsp, (6 + i) * wordSize));
	    __ bind(done);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「次に整数引数について, レジスタで渡す物を c_rarg{1,2,3,4,5} レジスタに移す.」
      ---------------------------------------- -}

	  // Now handle integrals.  Only do c_rarg1 if not static.
	  __ movl(c_rarg3, Address(rbx, methodOopDesc::access_flags_offset()));
	  __ testl(c_rarg3, JVM_ACC_STATIC);
	  __ cmovptr(Assembler::zero, c_rarg1, Address(rsp, 0));
	
	  __ movptr(c_rarg2, Address(rsp, wordSize));
	  __ movptr(c_rarg3, Address(rsp, 2 * wordSize));
	  __ movptr(c_rarg4, Address(rsp, 3 * wordSize));
	  __ movptr(c_rarg5, Address(rsp, 4 * wordSize));
	
  {- -------------------------------------------
  (1) コード生成:
      「SP の値を元に戻し, リターンする」
      ---------------------------------------- -}

	  // restore rsp
	  __ addptr(rsp, 14 * wordSize);
	
	  __ ret(0);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


