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
// Generic method entry to (asm) interpreter
//------------------------------------------------------------------------------------------------------------------------
//
```

### 名前(function name)
```
address InterpreterGenerator::generate_normal_entry(bool synchronized) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (G3 レジスタと G1 レジスタを作業用のレジスタとして使用)
      ---------------------------------------- -}

	  bool inc_counter  = UseCompiler || CountCompiledCalls;
	
	  // the following temporary registers are used during frame creation
	  const Register Gtmp1 = G3_scratch ;
	  const Register Gtmp2 = G1_scratch;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // make sure registers are different!
	  assert_different_registers(G2_thread, G5_method, Gargs, Gtmp1, Gtmp2);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Address size_of_parameters(G5_method, methodOopDesc::size_of_parameters_offset());
	  const Address size_of_locals    (G5_method, methodOopDesc::size_of_locals_offset());
	  // Seems like G5_method is live at the point this is used. So we could make this look consistent
	  // and use in the asserts.
	  const Address access_flags      (Lmethod,   methodOopDesc::access_flags_offset());
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(G5_method);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Register Glocals_size = G3;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Glocals_size, G4_scratch, Gframe_size);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      (access_flags を確認して not native & not abstract であることを確認している)
      ---------------------------------------- -}

	  // make sure method is not native & not abstract
	  // rethink these assertions - they can be simplified and shared (gri 2/25/2000)
	#ifdef ASSERT
	  __ ld(G5_method, methodOopDesc::access_flags_offset(), Gtmp1);
	  {
	    Label L;
	    __ btst(JVM_ACC_NATIVE, Gtmp1);
	    __ br(Assembler::zero, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ stop("tried to execute native method as non-native");
	    __ bind(L);
	  }
	  { Label L;
	    __ btst(JVM_ACC_ABSTRACT, Gtmp1);
	    __ br(Assembler::zero, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ stop("tried to execute abstract method as non-abstract");
	    __ bind(L);
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「TemplateInterpreterGenerator::generate_fixed_frame() が生成するコードで
        スタック上に新しいフレームを確保する」
      ---------------------------------------- -}

	  // generate the code to allocate the interpreter stack frame
	
	  generate_fixed_frame(false);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, FAST_DISPATCH が #define されている場合にのみ生成)
      「Interpreter::dispatch_table() の返値(= dispatch table のアドレス) を
        IdispatchTables レジスタにセットしておく」
      ---------------------------------------- -}

	#ifdef FAST_DISPATCH
	  __ set((intptr_t)Interpreter::dispatch_table(), IdispatchTables);
	                                          // set bytecode dispatch table base
	#endif
	
  {- -------------------------------------------
  (1) コード生成: 
      「局所変数用の領域を 0 クリアしておく」
  
      (以下, 呼び出し先のメソッドの "size_of_locals - size_of_parameters" 分の領域を 0 クリアする.
       クリア処理は bind( clear_loop ) から brx までのループで 1ワードずつ st_ptr することにより実行.)
      ---------------------------------------- -}

	  //
	  // Code to initialize the extra (i.e. non-parm) locals
	  //
	  Register init_value = noreg;    // will be G0 if we must clear locals
	  // The way the code was setup before zerolocals was always true for vanilla java entries.
	  // It could only be false for the specialized entries like accessor or empty which have
	  // no extra locals so the testing was a waste of time and the extra locals were always
	  // initialized. We removed this extra complication to already over complicated code.
	
	  init_value = G0;
	  Label clear_loop;
	
	  // NOTE: If you change the frame layout, this code will need to
	  // be updated!
	  __ lduh( size_of_locals, O2 );
	  __ lduh( size_of_parameters, O1 );
	  __ sll( O2, Interpreter::logStackElementSize, O2);
	  __ sll( O1, Interpreter::logStackElementSize, O1 );
	  __ sub( Llocals, O2, O2 );
	  __ sub( Llocals, O1, O1 );
	
	  __ bind( clear_loop );
	  __ inc( O2, wordSize );
	
	  __ cmp( O2, O1 );
	  __ brx( Assembler::lessEqualUnsigned, true, Assembler::pt, clear_loop );
	  __ delayed()->st_ptr( init_value, O2, 0 );
	
  {- -------------------------------------------
  (1) コード生成: 
      「カレントスレッドに対応する JavaThread の
        JavaThread::do_not_unlock_if_synchronized フィールドを, true にしておく」
      
      (synchronized メソッドの場合でも, lock_method() を呼ぶのはこの後なので, 
       この時点ではまだ BasicObjectLock を確保していない. 
  
       が, stack banging によって StackOverflowError が出ると, 例外処理で stack の巻き戻しが起こる可能性があり, 
       何もしないと (synchronized メソッドなので) unlock_if_synchronized_method() で解放処理が行われてしまう.
  
       そこで, thread の do_not_unlock_if_synchronized ビットを立てておく.
       これを立てておくと unlock_if_synchronized_method() 時に, このメソッドは見逃されるようになる.
       See: [here](no3059F5A.html) for details)
      ---------------------------------------- -}

	  const Address do_not_unlock_if_synchronized(G2_thread,
	    JavaThread::do_not_unlock_if_synchronized_offset());
	  // Since at this point in the method invocation the exception handler
	  // would try to exit the monitor of synchronized methods which hasn't
	  // been entered yet, we set the thread local variable
	  // _do_not_unlock_if_synchronized to true. If any exception was thrown by
	  // runtime, exception handling i.e. unlock_if_synchronized_method will
	  // check this thread local flag.
	  __ movbool(true, G3_scratch);
	  __ stbool(G3_scratch, do_not_unlock_if_synchronized);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, JIT コンパイルのためにメソッドの呼び出し回数を数える必要がある場合のみ生成する. 
                 より具体的には, UseCompiler オプションまたは CountCompiledCalls オプションが指定されている場合のみ生成)
  
      「InterpreterGenerator::generate_counter_incr() が生成するコードで, 
        invocation counter 値の増加処理と値のチェック処理を行う.
   
        値に応じて, 以下の3通りに分岐.
        * JIT コンパイル対象になる閾値を超えている場合:
          invocation_counter_overflow ラベルまでジャンプ
        * 監視対象になる閾値を超えているが, まだ mdp が確保されていない場合:
          profile_method ラベルまでジャンプ
        * それ以外の場合:
          このまま以下の処理にフォールスルー」
  
        (See: no3718SNC)
      ---------------------------------------- -}

	  // increment invocation counter and check for overflow
	  //
	  // Note: checking for negative value instead of overflow
	  //       so we have a 'sticky' overflow test (may be of
	  //       importance as soon as we have true MT/MP)
	  Label invocation_counter_overflow;
	  Label profile_method;
	  Label profile_method_continue;
	  Label Lcontinue;
	  if (inc_counter) {
	    generate_counter_incr(&invocation_counter_overflow, &profile_method, &profile_method_continue);
	    if (ProfileInterpreter) {
	      __ bind(profile_method_continue);
	    }
	  }

  {- -------------------------------------------
  (1) コード生成:
      「(ここが Lcontinue ラベルの位置)」
      ---------------------------------------- -}

	  __ bind(Lcontinue);
	
  {- -------------------------------------------
  (1) コード生成:
      「AbstractInterpreterGenerator::bang_stack_shadow_pages() が生成するコードで
        stack overflow をチェックする. (stack banging コード)」
       (See: [here](no3059d9W.html) for details)
      ---------------------------------------- -}

	  bang_stack_shadow_pages(false);
	
  {- -------------------------------------------
  (1) コード生成:
      「stack banging が終わったので, do_not_unlock_if_synchronized ビットを元に戻す」
      ---------------------------------------- -}

	  // reset the _do_not_unlock_if_synchronized flag
	  __ stbool(G0, do_not_unlock_if_synchronized);
	
  {- -------------------------------------------
  (1) コード生成: (引数で synchronized method だと指定されている場合にのみ生成)
      「InterpreterGenerator::lock_method() が生成するコードで, ロックを取得する」
      (See: [here](no9662EYy.html) for details)
      ---------------------------------------- -}

	  // check for synchronized methods
	  // Must happen AFTER invocation_counter check and stack overflow check,
	  // so method is not locked if overflows.
	
	  if (synchronized) {
	    lock_method();

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理) (引数で synchronized method だと指定されていない場合にのみ生成)
      (本当に synchronized でないことを確認するコードを生成している)
      ---------------------------------------- -}

	  } else {
	#ifdef ASSERT
	    { Label ok;
	      __ ld(access_flags, O0);
	      __ btst(JVM_ACC_SYNCHRONIZED, O0);
	      __ br( Assembler::zero, false, Assembler::pt, ok);
	      __ delayed()->nop();
	      __ stop("method needs synchronization");
	      __ bind(ok);
	    }
	#endif // ASSERT
	  }
	
  {- -------------------------------------------
  (1) (ここから先が実際の呼び出し処理)
      ---------------------------------------- -}

	  // start execution
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_thread();
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  // jvmti support
	  __ notify_method_entry();
	
  {- -------------------------------------------
  (1) コード生成:
      「dispatch_next() が生成するコードで, 
       メソッドの最初のバイトコードに対応するテンプレートへとジャンプする」
      ---------------------------------------- -}

	  // start executing instructions
	  __ dispatch_next(vtos);
	
	
  {- -------------------------------------------
  (1) (以下は, invocation counter の値が閾値を超えていた際の処理パス)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) コード生成: (ただし, profile_method ラベルによる飛び先については
                 JIT コンパイルのためにメソッドの呼び出し回数を数える必要がある場合のみ生成する. 
                 より具体的には, UseCompiler オプションまたは CountCompiledCalls オプションが指定されている場合のみ生成)
      「それぞれのラベル箇所では以下の処理が行われる.
        * invocation_counter_overflow ラベルによる飛び先
          InterpreterGenerator::generate_counter_overflow() が生成するコードにより, 
          JIT コンパイラを呼び出す (See: ...#TODO)
          その後, Lcontinue ラベルに戻って実行を再開する.
        * profile_method ラベルによる飛び先
          InterpreterRuntime::profile_method() を呼び出して,
          新しい methodDataOop オブジェクトを methodOop 内にセットする.
          その後, profile_method_continue ラベルに戻って実行を再開する.」
      ---------------------------------------- -}

	  if (inc_counter) {
	    if (ProfileInterpreter) {
	      // We have decided to profile this method in the interpreter
	      __ bind(profile_method);
	
	      __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::profile_method));
	      __ set_method_data_pointer_for_bcp();
	      __ ba(false, profile_method_continue);
	      __ delayed()->nop();
	    }
	
	    // handle invocation counter overflow
	    __ bind(invocation_counter_overflow);
	    generate_counter_overflow(Lcontinue);
	  }
	
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


