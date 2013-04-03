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
void TemplateInterpreterGenerator::generate_and_dispatch(Template* t, TosState tos_out) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コードの先頭箇所に, 必要に応じてデバッグ用のコードを生成しておく.
      (See: BytecodeCounter, BytecodeHistogram, BytecodePairHistogram)
      (See: BytecodeTracer)
      ---------------------------------------- -}

	  if (PrintBytecodeHistogram)                                    histogram_bytecode(t);
	#ifndef PRODUCT
	  // debugging code
	  if (CountBytecodes || TraceBytecodes || StopInterpreterAt > 0) count_bytecode();
	  if (PrintBytecodePairHistogram)                                histogram_bytecode_pair(t);
	  if (TraceBytecodes)                                            trace_bytecode(t);
	  if (StopInterpreterAt > 0)                                     stop_interpreter_at();
	  __ verify_FPU(1, t->tos_in());
	#endif // !PRODUCT

  {- -------------------------------------------
  (1) もし生成するテンプレートコード内に次のバイトコードに遷移する処理(ディスパッチ処理)が含まない場合には 
      (= Template::does_dispatch() が false の場合には),
      InterpreterMacroAssembler::dispatch_prolog() で
      ディスパッチ用の処理コード(テンプレート入り口部分用)を生成しておく.
  
      (なお Template::does_dispatch() が true になるのは, 
       goto や jsr, invoke 系のバイトコード, return 系のバイトコード, 等.
       詳細は, TemplateTable::initialize() 内での Template オブジェクト生成箇所の disp 欄を確認のこと.
  
       See: Template::initialize())
      ---------------------------------------- -}

	  int step;
	  if (!t->does_dispatch()) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    step = t->is_wide() ? Bytecodes::wide_length_for(t->bytecode()) : Bytecodes::length_for(t->bytecode());
	    if (tos_out == ilgl) tos_out = t->tos_out();
	    // compute bytecode size
	    assert(step > 0, "just checkin'");
	    // setup stuff for dispatching next bytecode

    {- -------------------------------------------
  (1.1) (#ifdef ASSERT で, かつ ProfileInterpreter や.. #TODO であれば, MDP をチェックするコードを生成)
        (#ifdef ASSERT 時にしか処理は生成されない.
         See: InterpreterMacroAssembler::verify_method_data_pointer())
        ---------------------------------------- -}

	    if (ProfileInterpreter && VerifyDataPointer
	        && methodDataOopDesc::bytecode_has_profile(t->bytecode())) {
	      __ verify_method_data_pointer();
	    }

    {- -------------------------------------------
  (1.1) InterpreterMacroAssembler::dispatch_prolog() で, テンプレートの入り口用のコードを生成
        ---------------------------------------- -}

	    __ dispatch_prolog(tos_out, step);
	  }

  {- -------------------------------------------
  (1) Template::generate() で, 各バイトコードの処理を行うためのコードを生成する.
      ---------------------------------------- -}

	  // generate template
	  t->generate(_masm);

  {- -------------------------------------------
  (1) もし生成するテンプレートコード内に次のバイトコードに遷移する処理(ディスパッチ処理)が含まない場合には 
      (= Template::does_dispatch() が false の場合),
      InterpreterMacroAssembler::dispatch_epilog() で
      ディスパッチ用の処理コード(テンプレート出口部分用)を生成しておく.
      
      (なお, Template::does_dispatch() が true の場合には, ここには到達しない(はず).
       #ifdef ASSERT 時には, 念のため should_not_reach_here() を埋めている)
      ---------------------------------------- -}

	  // advance
	  if (t->does_dispatch()) {
	#ifdef ASSERT
	    // make sure execution doesn't go beyond this point if code is broken
	    __ should_not_reach_here();
	#endif // ASSERT
	  } else {
	    // dispatch to next bytecode
	    __ dispatch_epilog(tos_out, step);
	  }
	}
	
```


