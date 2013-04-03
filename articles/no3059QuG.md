---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interpreter_sparc.cpp
### 説明(description)

```
#else
// LP64 passes floating point arguments in F1, F3, F5, etc. instead of
// O0, O1, O2 etc..
// Doubles are passed in D0, D2, D4
// We store the signature of the first 16 arguments in the first argument
// slot because it will be overwritten prior to calling the native
// function, with the pointer to the JNIEnv.
// If LP64 there can be up to 16 floating point arguments in registers
// or 6 integer registers.
```

### 名前(function name)
```
address AbstractInterpreterGenerator::generate_slow_signature_handler() {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  enum {
	    non_float  = 0,
	    float_sig  = 1,
	    double_sig = 2,
	    sig_mask   = 3
	  };
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Argument argv(0, true);
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) を設定する.」 #TODO
      ---------------------------------------- -}

	  // We are in the jni transition frame. Save the last_java_frame corresponding to the
	  // outer interpreter frame
	  //
	  __ set_last_Java_frame(FP, noreg);

  {- -------------------------------------------
  (1) コード生成:
      「レジスタを待避する 」
      (なお, G2_thread も待避対象)
      (また, Lmethod や Llocals の値は引数として使うので一時的に G レジスタにコピー)
  
      (mov(O7, I7) は, フレームを辿る場合用に, 
      ここで積んだフレームが正しいリターンアドレスを持っているように見せかける処理?? #TODO)
      ---------------------------------------- -}

	  // make sure the interpreter frame we've pushed has a valid return pc
	  __ mov(O7, I7);
	  __ mov(Lmethod, G3_scratch);
	  __ mov(Llocals, G4_scratch);
	  __ save_frame(0);
	  __ mov(G2_thread, L7_thread_cache);

  {- -------------------------------------------
  (1) コード生成:
      「引数をセットしつつ, InterpreterRuntime::slow_signature_handler() を呼び出す.」
  
      なお, 呼び出し時の引数は以下の通り.
      * 第1引数(O0): G2_thread の値
      * 第2引数(O1): Lmethod の値
      * 第3引数(O2): Llocals の値
      * 第4引数(O3): argv.address_in_frame() の値
      ---------------------------------------- -}

	  __ add(argv.address_in_frame(), O3);
	  __ mov(G2_thread, O0);
	  __ mov(G3_scratch, O1);
	  __ call(CAST_FROM_FN_PTR(address, InterpreterRuntime::slow_signature_handler), relocInfo::runtime_call_type);
	  __ delayed()->mov(G4_scratch, O2);

  {- -------------------------------------------
  (1) コード生成:
      「退避していた G2_thread を復帰させる」
      ---------------------------------------- -}

	  __ mov(L7_thread_cache, G2_thread);

  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) をクリアする」
      ---------------------------------------- -}

	  __ reset_last_Java_frame();
	
	
  {- -------------------------------------------
  (1) (この段階で, InterpreterRuntime::slow_signature_handler() によって
       引数は全てスタック上にコピーされている.
       とはいえ, スタックではなくレジスタで渡す引数もあるので, 
       以下の処理でレジスタで渡すものについてはレジスタに移しておく.
       
       なお, InterpreterRuntime::slow_signature_handler() の呼び出し後には, 
       Argument 0 の場所に signature を表す bit array が格納されている.)
      ---------------------------------------- -}

	  // load the register arguments (the C code packed them as varargs)
	  Address Sig = argv.address_in_frame();        // Argument 0 holds the signature
	  __ ld_ptr( Sig, G3_scratch );                   // Get register argument signature word into G3_scratch
	  __ mov( G3_scratch, G4_scratch);
	  __ srl( G4_scratch, 2, G4_scratch);             // Skip Arg 0
	  Label done;
	  for (Argument ldarg = argv.successor(); ldarg.is_float_register(); ldarg = ldarg.successor()) {
	    Label NonFloatArg;
	    Label LoadFloatArg;
	    Label LoadDoubleArg;
	    Label NextArg;
	    Address a = ldarg.address_in_frame();
	    __ andcc(G4_scratch, sig_mask, G3_scratch);
	    __ br(Assembler::zero, false, Assembler::pt, NonFloatArg);
	    __ delayed()->nop();
	
	    __ cmp(G3_scratch, float_sig );
	    __ br(Assembler::equal, false, Assembler::pt, LoadFloatArg);
	    __ delayed()->nop();
	
	    __ cmp(G3_scratch, double_sig );
	    __ br(Assembler::equal, false, Assembler::pt, LoadDoubleArg);
	    __ delayed()->nop();
	
	    __ bind(NonFloatArg);
	    // There are only 6 integer register arguments!
	    if ( ldarg.is_register() )
	      __ ld_ptr(ldarg.address_in_frame(), ldarg.as_register());
	    else {
	    // Optimization, see if there are any more args and get out prior to checking
	    // all 16 float registers.  My guess is that this is rare.
	    // If is_register is false, then we are done the first six integer args.
	      __ tst(G4_scratch);
	      __ brx(Assembler::zero, false, Assembler::pt, done);
	      __ delayed()->nop();
	
	    }
	    __ ba(false, NextArg);
	    __ delayed()->srl( G4_scratch, 2, G4_scratch );
	
	    __ bind(LoadFloatArg);
	    __ ldf( FloatRegisterImpl::S, a, ldarg.as_float_register(), 4);
	    __ ba(false, NextArg);
	    __ delayed()->srl( G4_scratch, 2, G4_scratch );
	
	    __ bind(LoadDoubleArg);
	    __ ldf( FloatRegisterImpl::D, a, ldarg.as_double_register() );
	    __ ba(false, NextArg);
	    __ delayed()->srl( G4_scratch, 2, G4_scratch );
	
	    __ bind(NextArg);
	
	  }
	
	  __ bind(done);

  {- -------------------------------------------
  (1) コード生成:
      「restore 命令でレジスタを復帰(Lscratch レジスタの値だけは現在の O0 の値に変更)させつつ, リターンする」
      ---------------------------------------- -}

	  __ ret();
	  __ delayed()->
	     restore(O0, 0, Lscratch);  // caller's Lscratch gets the result handler

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	#endif
	
```


