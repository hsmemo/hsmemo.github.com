---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.cpp

### 名前(function name)
```
void TemplateInterpreterGenerator::generate_all() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) AbstractInterpreterGenerator::generate_all() で, 
      ネイティブメソッド用の generic (slow) signature handler を生成.
      (See: [here](no3059asZ.html) for details)
      ---------------------------------------- -}

	  AbstractInterpreterGenerator::generate_all();
	
  {- -------------------------------------------
  (1) (以下, 様々な handler コードを生成していく)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_error_exit() を呼んで, 
      実行時エラー用のコード (TOS 状態がおかしい時などに使用するコード) を生成
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "error exits");
	    _unimplemented_bytecode    = generate_error_exit("unimplemented bytecode");
	    _illegal_bytecode_sequence = generate_error_exit("illegal bytecode sequence - method not verified");
	  }
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (トレース出力用の処理)  (See: BytecodeTracer)
      (TemplateInterpreterGenerator::generate_trace_code() を呼んで
       バイトコードトレース用コードを生成し, Interpreter::_trace_code に格納している.
       TOS に応じた9種類(btos,ctos,...)全てを生成)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (TraceBytecodes) {
	    CodeletMark cm(_masm, "bytecode tracing support");
	    Interpreter::_trace_code =
	      EntryPoint(
	        generate_trace_code(btos),
	        generate_trace_code(ctos),
	        generate_trace_code(stos),
	        generate_trace_code(atos),
	        generate_trace_code(itos),
	        generate_trace_code(ltos),
	        generate_trace_code(ftos),
	        generate_trace_code(dtos),
	        generate_trace_code(vtos)
	      );
	  }
	#endif // !PRODUCT
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_return_entry_for() を呼んで, 
      return entry 用の codelet を作成
  
      (これは, メソッド呼び出しから返ってきた後で, その後片付け的な作業を行うコード.
       See: [here](no3059YlB.html) for details)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "return entry points");
	    for (int i = 0; i < Interpreter::number_of_return_entries; i++) {
	      Interpreter::_return_entry[i] =
	        EntryPoint(
	          generate_return_entry_for(itos, i),
	          generate_return_entry_for(itos, i),
	          generate_return_entry_for(itos, i),
	          generate_return_entry_for(atos, i),
	          generate_return_entry_for(itos, i),
	          generate_return_entry_for(ltos, i),
	          generate_return_entry_for(ftos, i),
	          generate_return_entry_for(dtos, i),
	          generate_return_entry_for(vtos, i)
	        );
	    }
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_earlyret_entry_for() を呼んで,
      JVMTI の ForceEarlyReturn 処理用のコードレットを生成.
      (See: [here](no3059azN.html) for details)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "earlyret entry points");
	    Interpreter::_earlyret_entry =
	      EntryPoint(
	        generate_earlyret_entry_for(btos),
	        generate_earlyret_entry_for(ctos),
	        generate_earlyret_entry_for(stos),
	        generate_earlyret_entry_for(atos),
	        generate_earlyret_entry_for(itos),
	        generate_earlyret_entry_for(ltos),
	        generate_earlyret_entry_for(ftos),
	        generate_earlyret_entry_for(dtos),
	        generate_earlyret_entry_for(vtos)
	      );
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_deopt_entry_for() を呼んで, 
      Deoptimization 処理で使われる deopt entry を生成.
      (See: Deoptimization)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "deoptimization entry points");
	    for (int i = 0; i < Interpreter::number_of_deopt_entries; i++) {
	      Interpreter::_deopt_entry[i] =
	        EntryPoint(
	          generate_deopt_entry_for(itos, i),
	          generate_deopt_entry_for(itos, i),
	          generate_deopt_entry_for(itos, i),
	          generate_deopt_entry_for(atos, i),
	          generate_deopt_entry_for(itos, i),
	          generate_deopt_entry_for(ltos, i),
	          generate_deopt_entry_for(ftos, i),
	          generate_deopt_entry_for(dtos, i),
	          generate_deopt_entry_for(vtos, i)
	        );
	    }
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_result_handler_for() を呼んで, 
      ネイティブメソッド用の result handler を作成
      (See: [here](no3059asZ.html) for details)
  
      (なお, number_of_result_handlers は 10.
       Bool,Byte,Char,Int,Short,Long,Float,Double,Object,VOID の 10種類を意味する.
       See: AbstractInterpreter::number_of_result_handlers)
  
      (!is_generated の所は何を判定している?? 既に作成済みのケースがあるのか?? #TODO)
  
      (作成したコードは Interpreter::_native_abi_to_tosca[] に格納.
       AbstractInterpreter::result_handler() や
       CppInterpreter::native_result_to_tosca() でアクセスできる.
       が, CppInterpreter::native_result_to_tosca() の方はどこからも使われていない...
       See: AbstractInterpreter::result_handler(), CppInterpreter::native_result_to_tosca())
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "result handlers for native calls");
	    // The various result converter stublets.
	    int is_generated[Interpreter::number_of_result_handlers];
	    memset(is_generated, 0, sizeof(is_generated));
	
	    for (int i = 0; i < Interpreter::number_of_result_handlers; i++) {
	      BasicType type = types[i];
	      if (!is_generated[Interpreter::BasicType_as_index(type)]++) {
	        Interpreter::_native_abi_to_tosca[Interpreter::BasicType_as_index(type)] = generate_result_handler_for(type);
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) Interpreter::return_entry() を呼んで, 
      invokevirtual/invokeinterface 用の return entry を取得する.
  
      (なお, Interpreter::return_entry() は TemplateInterpreter::return_entry() のこと.
       上の TemplateInterpreterGenerator::generate_return_entry_for() 呼び出しで生成した
       return entry を取得する関数であって, ここでまた新しく作るわけではない.
  
       なお, TOS 状態毎に取得する.
       生成結果はそれぞれ, Interpreter::_return_3_addrs_by_index 及び 
       Interpreter::_return_5_addrs_by_index フィールドにある配列に格納.)
    
      (メソッド呼び出し系のバイトコード(invoke*)は長さが二種類あるため, 
       呼び出しから返ってきた後で 3byte 先のバイトコードに dispatch しないといけないケースと
       5byte 先のバイトコードを dispatch しないといけないケースの 2種類がある.
       それぞれに対応できるよう, 両方の種類の return entry を初期化している.
       See: [here](no3059lvH.html) for details)
      ---------------------------------------- -}

	  for (int j = 0; j < number_of_states; j++) {
	    const TosState states[] = {btos, ctos, stos, itos, ltos, ftos, dtos, atos, vtos};
	    int index = Interpreter::TosState_as_index(states[j]);
	    Interpreter::_return_3_addrs_by_index[index] = Interpreter::return_entry(states[j], 3);
	    Interpreter::_return_5_addrs_by_index[index] = Interpreter::return_entry(states[j], 5);
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_continuation_for() を呼んで, ...#TODO
  
      (これは何だ??
       というか, そもそもこの Interpreter::_continuation_entry はどこで使われている?? #TODO)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "continuation entry points");
	    Interpreter::_continuation_entry =
	      EntryPoint(
	        generate_continuation_for(btos),
	        generate_continuation_for(ctos),
	        generate_continuation_for(stos),
	        generate_continuation_for(atos),
	        generate_continuation_for(itos),
	        generate_continuation_for(ltos),
	        generate_continuation_for(ftos),
	        generate_continuation_for(dtos),
	        generate_continuation_for(vtos)
	      );
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_safept_entry_for() を呼んで, 
      Template Interpreter を Safepoint 停止させるためのコードを生成する.
  
      (これは, このメソッドの末尾で呼び出されている TemplateInterpreterGenerator::set_safepoints_for_all_bytes() 内で
       Safepoint 停止用の dispatch table (TemplateInterpreter::_safept_table) を構築するために使用されている.
       後述)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "safepoint entry points");
	    Interpreter::_safept_entry =
	      EntryPoint(
	        generate_safept_entry_for(btos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(ctos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(stos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(atos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(itos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(ltos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(ftos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(dtos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint)),
	        generate_safept_entry_for(vtos, CAST_FROM_FN_PTR(address, InterpreterRuntime::at_safepoint))
	      );
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_throw_exception() を呼んで, 
      例外発生時に例外送出やスタックの巻戻しを行うためのコードを生成.
      (See: [here](no30593YX.html) for details)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "exception handling");
	    // (Note: this is not safepoint safe because thread may return to compiled code)
	    generate_throw_exception();
	  }
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_${例外名}_handler() を呼んで, 
      Java コード実行中に athrow 以外で発生する可能性のある例外について, 
      特注の例外ハンドリング用(例外送出用)エントリポイントコードを生成.
      (See: [here](no3059y5N.html) for details)
      
      (なお, generate_exception_handler() と generate_klass_exception_handler() だけは share 部で定義されている 
       とはいえ, generate_exception_handler_common() を呼び出すだけなので, cpu/ 下に戻ってくるが...)
      ---------------------------------------- -}

	  { CodeletMark cm(_masm, "throw exception entrypoints");
	    Interpreter::_throw_ArrayIndexOutOfBoundsException_entry = generate_ArrayIndexOutOfBounds_handler("java/lang/ArrayIndexOutOfBoundsException");
	    Interpreter::_throw_ArrayStoreException_entry            = generate_klass_exception_handler("java/lang/ArrayStoreException"                 );
	    Interpreter::_throw_ArithmeticException_entry            = generate_exception_handler("java/lang/ArithmeticException"           , "/ by zero");
	    Interpreter::_throw_ClassCastException_entry             = generate_ClassCastException_handler();
	    Interpreter::_throw_NullPointerException_entry           = generate_exception_handler("java/lang/NullPointerException"          , NULL       );
	    Interpreter::_throw_StackOverflowError_entry             = generate_StackOverflowError_handler();
	  }
	
	
	
  {- -------------------------------------------
  (1) AbstractInterpreterGenerator::generate_method_entry() を呼んで, 
      各種メソッドのエントリ処理用のコードを生成.
      (See: [here](no3059n2f.html) for details)
      
      (なお, この生成処理では method_entry() というマクロによってコードを簡潔化している)
      ---------------------------------------- -}

	#define method_entry(kind)                                                                    \
	  { CodeletMark cm(_masm, "method entry point (kind = " #kind ")");                    \
	    Interpreter::_entry_table[Interpreter::kind] = generate_method_entry(Interpreter::kind);  \
	  }
	
	  // all non-native method kinds
	  method_entry(zerolocals)
	  method_entry(zerolocals_synchronized)
	  method_entry(empty)
	  method_entry(accessor)
	  method_entry(abstract)
	  method_entry(method_handle)
	  method_entry(java_lang_math_sin  )
	  method_entry(java_lang_math_cos  )
	  method_entry(java_lang_math_tan  )
	  method_entry(java_lang_math_abs  )
	  method_entry(java_lang_math_sqrt )
	  method_entry(java_lang_math_log  )
	  method_entry(java_lang_math_log10)
	  method_entry(java_lang_ref_reference_get)
	
	  // all native method kinds (must be one contiguous block)
	  Interpreter::_native_entry_begin = Interpreter::code()->code_end();
	  method_entry(native)
	  method_entry(native_synchronized)
	  Interpreter::_native_entry_end = Interpreter::code()->code_end();
	
	#undef method_entry
	
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::set_entry_points_for_all_bytes() を呼んで, 
      各々のバイトコードに対応するテンプレートを生成.
      ---------------------------------------- -}

	  // Bytecodes
	  set_entry_points_for_all_bytes();

  {- -------------------------------------------
  (1) 最後に, TemplateInterpreterGenerator::set_safepoints_for_all_bytes() を呼んで, 
      Safepoint 停止用の dispatch table (TemplateInterpreter::_safept_table) を構築する.
      ---------------------------------------- -}

	  set_safepoints_for_all_bytes();
	}
	
```


