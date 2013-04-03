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
#ifndef _LP64
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
	  for (Argument ldarg = argv.successor(); ldarg.is_register(); ldarg = ldarg.successor()) {
	      __ ld_ptr(ldarg.address_in_frame(), ldarg.as_register());
	  }

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
	
```


