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
// increment invocation count & check for overflow
//
// Note: checking for negative value instead of overflow
//       so we have a 'sticky' overflow test
//
// Lmethod: method
// ??: invocation counter
//
```

### 名前(function name)
```
void InterpreterGenerator::generate_counter_incr(Label* overflow, Label* profile_method, Label* profile_method_continue) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数には, TieredCompilation オプションの値に応じて 2つの処理パスがある)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下が, TieredCompilation オプションが指定されている場合の処理)
      ---------------------------------------- -}

	  // Note: In tiered we increment either counters in methodOop or in MDO depending if we're profiling or not.
	  if (TieredCompilation) {

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    const int increment = InvocationCounter::count_increment;
	    const int mask = ((1 << Tier0InvokeNotifyFreqLog) - 1) << InvocationCounter::count_shift;
	    Label no_mdo, done;
	    if (ProfileInterpreter) {
	      // If no method data exists, go to profile_continue.
	      __ ld_ptr(Lmethod, methodOopDesc::method_data_offset(), G4_scratch);
	      __ br_null(G4_scratch, false, Assembler::pn, no_mdo);
	      __ delayed()->nop();
	      // Increment counter
	      Address mdo_invocation_counter(G4_scratch,
	                                     in_bytes(methodDataOopDesc::invocation_counter_offset()) +
	                                     in_bytes(InvocationCounter::counter_offset()));
	      __ increment_mask_and_jump(mdo_invocation_counter, increment, mask,
	                                 G3_scratch, Lscratch,
	                                 Assembler::zero, overflow);
	      __ ba(false, done);
	      __ delayed()->nop();
	    }
	
	    // Increment counter in methodOop
	    __ bind(no_mdo);
	    Address invocation_counter(Lmethod,
	                               in_bytes(methodOopDesc::invocation_counter_offset()) +
	                               in_bytes(InvocationCounter::counter_offset()));
	    __ increment_mask_and_jump(invocation_counter, increment, mask,
	                               G3_scratch, Lscratch,
	                               Assembler::zero, overflow);
	    __ bind(done);

  {- -------------------------------------------
  (1) (以下が, TieredCompilation オプションが指定されていない場合の処理)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) コード生成:
        「カウンタの値をインクリメントする」#TODO
        ---------------------------------------- -}

	    // Update standard invocation counters
	    __ increment_invocation_counter(O0, G3_scratch);
	    if (ProfileInterpreter) {  // %%% Merge this into methodDataOop
	      Address interpreter_invocation_counter(Lmethod,in_bytes(methodOopDesc::interpreter_invocation_counter_offset()));
	      __ ld(interpreter_invocation_counter, G3_scratch);
	      __ inc(G3_scratch);
	      __ st(G3_scratch, interpreter_invocation_counter);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成: (ただし, ProfileInterpreter オプションが指定されていない場合や, 引数の profile_method が NULL の場合には生成しない)
        「カウンタの値を閾値(InvocationCounter::InterpreterProfileLimitの値)と比較する.
         もしカウンタ値が閾値未満であれば, 引数で指定された profile_method_continue ラベルにジャンプする.
  
         そうでなければ, InterpreterMacroAssembler::test_method_data_pointer() が生成するコードによって
         ImethodDataPtr レジスタの値を調べる.
         まだ mdp が生成されてなければ, 引数で指定された profile_method ラベルにジャンプする」
        ---------------------------------------- -}

	    if (ProfileInterpreter && profile_method != NULL) {
	      // Test to see if we should create a method data oop
	      AddressLiteral profile_limit((address)&InvocationCounter::InterpreterProfileLimit);
	      __ load_contents(profile_limit, G3_scratch);
	      __ cmp(O0, G3_scratch);
	      __ br(Assembler::lessUnsigned, false, Assembler::pn, *profile_method_continue);
	      __ delayed()->nop();
	
	      // if no method data exists, go to profile_method
	      __ test_method_data_pointer(*profile_method);
	    }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「カウンタの値を閾値(InvocationCounter::InterpreterInvocationLimitの値)と比較する.
         もしカウンタ値が閾値以上であれば, 引数で指定された overflow ラベルにジャンプする.」
        ---------------------------------------- -}

	    AddressLiteral invocation_limit((address)&InvocationCounter::InterpreterInvocationLimit);
	    __ load_contents(invocation_limit, G3_scratch);
	    __ cmp(O0, G3_scratch);
	    __ br(Assembler::greaterEqualUnsigned, false, Assembler::pn, *overflow);
	    __ delayed()->nop();
	  }
	
	}
	
```


