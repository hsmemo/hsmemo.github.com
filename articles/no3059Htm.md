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
// increment invocation count & check for overflow
//
// Note: checking for negative value instead of overflow
//       so we have a 'sticky' overflow test
//
// rbx: method
// ecx: invocation counter
//
```

### 名前(function name)
```
void InterpreterGenerator::generate_counter_incr(
        Label* overflow,
        Label* profile_method,
        Label* profile_method_continue) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数には, TieredCompilation オプションの値に応じて 2つの処理パスがある)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Address invocation_counter(rbx, in_bytes(methodOopDesc::invocation_counter_offset()) +
	                                        in_bytes(InvocationCounter::counter_offset()));

  {- -------------------------------------------
  (1) (以下が, TieredCompilation オプションが指定されている場合の処理)
      ---------------------------------------- -}

	  // Note: In tiered we increment either counters in methodOop or in MDO depending if we're profiling or not.
	  if (TieredCompilation) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    int increment = InvocationCounter::count_increment;
	    int mask = ((1 << Tier0InvokeNotifyFreqLog)  - 1) << InvocationCounter::count_shift;
	    Label no_mdo, done;

    {- -------------------------------------------
  (1.1) コード生成: (ただし, ProfileInterpreter オプションが指定されていない場合には生成しない)
        「もし methodDataOopDesc(mdo) が付いていれば, 
          InterpreterMacroAssembler::increment_mask_and_jump() が生成するコードによって
          mdo の _invocation_counter フィールドのカウンタ値をインクリメントし, 閾値チェックを行う.
          閾値に達していた場合は, 引数で指定された overflow ラベルにジャンプする.
          そうでなければ, done ラベルまでジャンプする (= 今回のカウンタ値の処理は終了).
  
          もし methodDataOopDesc が付いていなければ, 
          no_mdo ラベルまでジャンプする」
        ---------------------------------------- -}

	    if (ProfileInterpreter) {
	      // Are we profiling?
	      __ movptr(rax, Address(rbx, methodOopDesc::method_data_offset()));
	      __ testptr(rax, rax);
	      __ jccb(Assembler::zero, no_mdo);
	      // Increment counter in the MDO
	      const Address mdo_invocation_counter(rax, in_bytes(methodDataOopDesc::invocation_counter_offset()) +
	                                                in_bytes(InvocationCounter::counter_offset()));
	      __ increment_mask_and_jump(mdo_invocation_counter, increment, mask, rcx, false, Assembler::zero, overflow);
	      __ jmpb(done);
	    }

    {- -------------------------------------------
  (1.1) コード生成:
        「(ここが no_mdo ラベルの位置)」
        ---------------------------------------- -}

	    __ bind(no_mdo);

    {- -------------------------------------------
  (1.1) コード生成:
        「InterpreterMacroAssembler::increment_mask_and_jump() が生成するコードによって
          実行中のメソッドに対応する methodOopDesc の 
          _invocation_counter フィールドのカウンタ値をインクリメントし, 閾値チェックを行う.
          閾値に達していた場合は, 引数で指定された overflow ラベルにジャンプする.
          そうでなければ, このままフォールスルーして, 今回のカウンタ値の処理は終了」
        ---------------------------------------- -}

	    // Increment counter in methodOop (we don't need to load it, it's in ecx).
	    __ increment_mask_and_jump(invocation_counter, increment, mask, rcx, true, Assembler::zero, overflow);

    {- -------------------------------------------
  (1.1) コード生成:
        「(ここが done ラベルの位置)」
        ---------------------------------------- -}

	    __ bind(done);

  {- -------------------------------------------
  (1) (以下が, TieredCompilation オプションが指定されていない場合の処理)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    const Address backedge_counter(rbx,
	                                   methodOopDesc::backedge_counter_offset() +
	                                   InvocationCounter::counter_offset());
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, ProfileInterpreter オプションが指定されていない場合には生成しない)
        「(プロファイル情報を取得する場合には)
          実行中のメソッドに対応する methodOopDesc の 
          _interpreter_invocation_count フィールドの値をインクリメントしておく」
        ---------------------------------------- -}

	    if (ProfileInterpreter) { // %%% Merge this into methodDataOop
	      __ incrementl(Address(rbx,
	                            methodOopDesc::interpreter_invocation_counter_offset()));
	    }

    {- -------------------------------------------
  (1.1) コード生成:
        「実行中のメソッドに対応する methodOopDesc の 
          _invocation_counter フィールドのカウンタ値をインクリメントする.
  
          (なおついでに, rcx レジスタに 
          _backedge_counter と _invocation_counter を足した値をセットしている)」
  
         (なお, このコードでは
         rcx に予め _invocation_counter フィールドのアドレスが入っていると想定している.
         この関数の先頭にあるコメントも参照)
        ---------------------------------------- -}

	    // Update standard invocation counters
	    __ movl(rax, backedge_counter);   // load backedge counter
	
	    __ incrementl(rcx, InvocationCounter::count_increment);
	    __ andl(rax, InvocationCounter::count_mask_value); // mask out the status bits
	
	    __ movl(invocation_counter, rcx); // save invocation count
	    __ addl(rcx, rax);                // add both counters
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, ProfileInterpreter オプションが指定されていない場合や, 引数の profile_method が NULL の場合には生成しない)
        「(プロファイル情報を取得する場合, ある程度実行回数が多いコードには methodDataOopDesc(mdo) を付ける.
          ここでカウンタ値をチェックし, カウンタ値が大きいのに mdo がまだ付いてなければ, 付けるコードにジャンプする)
  
         カウンタの値を閾値(InvocationCounter::InterpreterProfileLimitの値)と比較する.
         もしカウンタ値が閾値未満であれば, 引数で指定された profile_method_continue ラベルにジャンプする.
  
         そうでなければ, InterpreterMacroAssembler::test_method_data_pointer() が生成するコードによって
         ImethodDataPtr レジスタの値を調べる.
         まだ mdo が生成されてなければ, 引数で指定された profile_method ラベルにジャンプする」
    
  
         (なお, 引数の profile_method が NULL 以外の値を取るのは
          インタープリタで実行されているメソッドの場合のみ.
          つまり, "profile_method != NULL" という条件は「ネイティブメソッドではない」という意味)
        ---------------------------------------- -}

	    // profile_method is non-null only for interpreted method so
	    // profile_method != NULL == !native_call
	
	    if (ProfileInterpreter && profile_method != NULL) {
	      // Test to see if we should create a method data oop
	      __ cmp32(rcx, ExternalAddress((address)&InvocationCounter::InterpreterProfileLimit));
	      __ jcc(Assembler::less, *profile_method_continue);
	
	      // if no method data exists, go to profile_method
	      __ test_method_data_pointer(rax, *profile_method);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「カウンタの値を閾値(InvocationCounter::InterpreterInvocationLimitの値)と比較する.
         もしカウンタ値が閾値以上であれば, 引数で指定された overflow ラベルにジャンプする.」
        ---------------------------------------- -}

	    __ cmp32(rcx, ExternalAddress((address)&InvocationCounter::InterpreterInvocationLimit));
	    __ jcc(Assembler::aboveEqual, *overflow);
	  }
	}
	
```


