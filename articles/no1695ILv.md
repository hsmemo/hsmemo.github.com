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
#ifdef _WIN64
```

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
	  __ subptr(rsp, 4 * wordSize);

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
	  // rsp: 3 integer or float args (if static first is unused)
	  //      1 float/double identifiers
	  //        return address
	  //        stack args
	  //        garbage
	  //        expression stack bottom
	  //        bcp (NULL)
	  //        ...
	
  {- -------------------------------------------
  (1) コード生成:
      「レジスタで渡す引数はレジスタに移しておく.」
      ---------------------------------------- -}

	  // Do FP first so we can use c_rarg3 as temp
	  __ movl(c_rarg3, Address(rsp, 3 * wordSize)); // float/double identifiers
	
	  for ( int i= 0; i < Argument::n_int_register_parameters_c-1; i++ ) {
	    XMMRegister floatreg = as_XMMRegister(i+1);
	    Label isfloatordouble, isdouble, next;
	
	    __ testl(c_rarg3, 1 << (i*2));      // Float or Double?
	    __ jcc(Assembler::notZero, isfloatordouble);
	
	    // Do Int register here
	    switch ( i ) {
	      case 0:
	        __ movl(rscratch1, Address(rbx, methodOopDesc::access_flags_offset()));
	        __ testl(rscratch1, JVM_ACC_STATIC);
	        __ cmovptr(Assembler::zero, c_rarg1, Address(rsp, 0));
	        break;
	      case 1:
	        __ movptr(c_rarg2, Address(rsp, wordSize));
	        break;
	      case 2:
	        __ movptr(c_rarg3, Address(rsp, 2 * wordSize));
	        break;
	      default:
	        break;
	    }
	
	    __ jmp (next);
	
	    __ bind(isfloatordouble);
	    __ testl(c_rarg3, 1 << ((i*2)+1));     // Double?
	    __ jcc(Assembler::notZero, isdouble);
	
	// Do Float Here
	    __ movflt(floatreg, Address(rsp, i * wordSize));
	    __ jmp(next);
	
	// Do Double here
	    __ bind(isdouble);
	    __ movdbl(floatreg, Address(rsp, i * wordSize));
	
	    __ bind(next);
	  }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「SP の値を元に戻し, リターンする」
      ---------------------------------------- -}

	  // restore rsp
	  __ addptr(rsp, 4 * wordSize);
	
	  __ ret(0);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


