---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)
//
// Generate a fixed interpreter frame. This is identical setup for interpreted
// methods and for native methods hence the shared code.



### 名前(function name)
```
void TemplateInterpreterGenerator::generate_fixed_frame(bool native_call) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数が生成するコードは, 以下のような処理を行う)
      ---------------------------------------- -}

	  //
	  //
	  // The entry code sets up a new interpreter frame in 4 steps:
	  //
	  // 1) Increase caller's SP by for the extra local space needed:
	  //    (check for overflow)
	  //    Efficient implementation of xload/xstore bytecodes requires
	  //    that arguments and non-argument locals are in a contigously
	  //    addressable memory block => non-argument locals must be
	  //    allocated in the caller's frame.
	  //
	  // 2) Create a new stack frame and register window:
	  //    The new stack frame must provide space for the standard
	  //    register save area, the maximum java expression stack size,
	  //    the monitor slots (0 slots initially), and some frame local
	  //    scratch locations.
	  //
	  // 3) The following interpreter activation registers must be setup:
	  //    Lesp       : expression stack pointer
	  //    Lbcp       : bytecode pointer
	  //    Lmethod    : method
	  //    Llocals    : locals pointer
	  //    Lmonitors  : monitor pointer
	  //    LcpoolCache: constant pool cache
	  //
	  // 4) Initialize the non-argument locals if necessary:
	  //    Non-argument locals may need to be initialized to NULL
	  //    for GC to work. If the oop-map information is accurate
	  //    (in the absence of the JSR problem), no initialization
	  //    is necessary.
	  //
	  // (gri - 2/25/2000)
	
	
  {- -------------------------------------------
  (1) (変数宣言など)
  
      (max_stack は, 呼び出されたメソッドの最大オペランドスタック長(expression stack size))
  
      (extra_space はフレーム中に用意しないといけないプラットフォーム依存な領域の大きさ.
       以下の 3つ(ネイティブメソッドでない場合は 2つ)の合計値となっている.
       * frame::interpreter_frame_vm_local_words
         フレーム内でインタープリタが作業用に使用する領域の大きさ
       * frame::memory_parameter_word_sp_offset
         window save area と第6引数までを格納できるスペース量
         (Sparc の calling convention 通り)
       * frame::interpreter_frame_extra_outgoing_argument_words  
         (ネイティブメソッドでなければ, この値は足されない)
         ネイティブメソッドの場合に追加で渡すべき引数の数.
         class と JNIEnv の 2つ.                                       )
      ---------------------------------------- -}

	  const Address size_of_parameters(G5_method, methodOopDesc::size_of_parameters_offset());
	  const Address size_of_locals    (G5_method, methodOopDesc::size_of_locals_offset());
	  const Address max_stack         (G5_method, methodOopDesc::max_stack_offset());
	  int rounded_vm_local_words = round_to( frame::interpreter_frame_vm_local_words, WordsPerLong );
	
	  const int extra_space =
	    rounded_vm_local_words +                   // frame local scratch space
	    //6815692//methodOopDesc::extra_stack_words() +       // extra push slots for MH adapters
	    frame::memory_parameter_word_sp_offset +   // register save area
	    (native_call ? frame::interpreter_frame_extra_outgoing_argument_words : 0);
	
	  const Register Glocals_size = G3;
	  const Register Otmp1 = O3;
	  const Register Otmp2 = O4;

  {- -------------------------------------------
  (1) (なお, call_stub が使ってしまっているため Lscratch は一時レジスタとしては使えない, とのこと)
      ---------------------------------------- -}

	  // Lscratch can't be used as a temporary because the call_stub uses
	  // it to assert that the stack frame was setup correctly.
	
  {- -------------------------------------------
  (1) (まず, 必要な局所変数領域の大きさを計算し, その分の領域を確保する処理を行う. 
       この際, 必要であれば呼び出し元の SP をずらす処理を行う)
  
      (局所変数領域は引数を含むので, 呼び出し元のフレーム内に確保する必要がある.
       詳細はフレーム構造を参照.)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) コード生成:
        「引数の数を取得する」
        ---------------------------------------- -}

	  __ lduh( size_of_parameters, Glocals_size);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「Gargs の値を修正しておく」
  
        (この時点では最後の引数を指しているが, 正しくは最初の引数を指すべき.
         このため, 引数分だけ増加させておく.)
        ---------------------------------------- -}

	  // Gargs points to first local + BytesPerWord
	  // Set the saved SP after the register window save
	  //
	  assert_different_registers(Gargs, Glocals_size, Gframe_size, O5_savedSP);
	  __ sll(Glocals_size, Interpreter::logStackElementSize, Otmp1);
	  __ add(Gargs, Otmp1, Gargs);
	
    {- -------------------------------------------
  (1.1) (以下で, 新しく確保するスタックフレームの大きさを計算して Gframe_size に格納する.
  
         この処理は, ネイティブメソッドかどうかで生成するコードが異なる.
         ネイティブメソッドでない場合には, 呼び出し元の SP をずらす処理も行う.)
        ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) コード生成: (これは, ネイティブメソッドの場合のコード)
        「新しく確保するスタックフレームの大きさを計算する.
          結果は Gframe_size に格納.」
  
         (なお, ここでは大きさは以下の用に計算している.
           
            スタックフレームの大きさ = round_to(レジスタに載らない引数の数 + extra_space) * BytesPerWord)
        ---------------------------------------- -}

	  if (native_call) {
	    __ calc_mem_param_words( Glocals_size, Gframe_size );
	    __ add( Gframe_size,  extra_space, Gframe_size);
	    __ round_to( Gframe_size, WordsPerLong );
	    __ sll( Gframe_size, LogBytesPerWord, Gframe_size );
	  } else {
	
    {- -------------------------------------------
  (1.1) コード生成: (これは, ネイティブメソッドでない場合のコード)
        「引数分を引いた局所変数領域の大きさを計算しておく.
          結果は Glocals_size に格納」
        ---------------------------------------- -}

	    //
	    // Compute number of locals in method apart from incoming parameters
	    //
	    __ lduh( size_of_locals, Otmp1 );
	    __ sub( Otmp1, Glocals_size, Glocals_size );
	    __ round_to( Glocals_size, WordsPerLong );
	    __ sll( Glocals_size, Interpreter::logStackElementSize, Glocals_size );
	
    {- -------------------------------------------
  (1.1) コード生成: (これは, ネイティブメソッドでない場合のコード)
        「新しく確保するスタックフレームの大きさを計算する.
          結果は Gframe_size に格納.」
  
         (なお, ここでは大きさは以下の用に計算している.
           
            round_to(max_stack + extra_space) * BytesPerWord)
        ---------------------------------------- -}

	    // see if the frame is greater than one page in size. If so,
	    // then we need to verify there is enough stack space remaining
	    // Frame_size = (max_stack + extra_space) * BytesPerWord;
	    __ lduh( max_stack, Gframe_size );
	    __ add( Gframe_size, extra_space, Gframe_size );
	    __ round_to( Gframe_size, WordsPerLong );
	    __ sll( Gframe_size, Interpreter::logStackElementSize, Gframe_size);
	
    {- -------------------------------------------
  (1.1) コード生成: (これは, ネイティブメソッドでない場合のコード)
        「InterpreterGenerator::generate_stack_overflow_check() が生成するコードにより, 
         stack overflow のチェックを行う.
         もしスタックに空きがなければ, StackOverflowError.」
    
        (なお, この処理の間だけ Gframe_size の値を Glocals_size 分だけ増加させている.
         これは, この後で Glocals_size 分だけ SP をずらすことから, 
         実質的には Gframe_size + Glocals_size 分だけ確保することになるため.
         stack banging が終わったら, また元の値に戻している)
        ---------------------------------------- -}

	    // Add in java locals size for stack overflow check only
	    __ add( Gframe_size, Glocals_size, Gframe_size );
	
	    const Register Otmp2 = O4;
	    assert_different_registers(Otmp1, Otmp2, O5_savedSP);
	    generate_stack_overflow_check(Gframe_size, Otmp1, Otmp2);
	
	    __ sub( Gframe_size, Glocals_size, Gframe_size);
	
    {- -------------------------------------------
  (1.1) コード生成: (これは, ネイティブメソッドでない場合のコード)
        「(引数を除いた)局所変数分だけ, 呼び出し元の SP の値をずらしておく」
  
         (なお, 当然ながら O5_savedSP はずらさない)
        ---------------------------------------- -}

	    //
	    // bump SP to accomodate the extra locals
	    //
	    __ sub( SP, Glocals_size, SP );
	  }

  {- -------------------------------------------
  (1) (ここまでが, 局所変数領域の大きさの計算処理, 及び局所変数領域の確保処理)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) コード生成:
      「save 命令を使ってフレームを確保する(= SPをずらす)
        (同時に, レジスタの退避も行う)」
    
       (確保量は, 上で計算した Gframe_size 分)
      ---------------------------------------- -}

	  //
	  // now set up a stack frame with the size computed above
	  //
	  __ neg( Gframe_size );
	  __ save( SP, Gframe_size, SP );
	
  {- -------------------------------------------
  (1) コード生成:
      「Template Interpreter の実行に必要なレジスタの値をセットする.
        (Lesp, Lbcp, Lmethod, Llocals, Lmonitors, LcpoolCache)」
      ---------------------------------------- -}

	  //
	  // now set up all the local cache registers
	  //
	  // NOTE: At this point, Lbyte_code/Lscratch has been modified. Note
	  // that all present references to Lbyte_code initialize the register
	  // immediately before use
	  if (native_call) {
	    __ mov(G0, Lbcp);
	  } else {
	    __ ld_ptr(G5_method, methodOopDesc::const_offset(), Lbcp);
	    __ add(Lbcp, in_bytes(constMethodOopDesc::codes_offset()), Lbcp);
	  }
	  __ mov( G5_method, Lmethod);                 // set Lmethod
	  __ get_constant_pool_cache( LcpoolCache );   // set LcpoolCache
	  __ sub(FP, rounded_vm_local_words * BytesPerWord, Lmonitors ); // set Lmonitors
	#ifdef _LP64
	  __ add( Lmonitors, STACK_BIAS, Lmonitors );   // Account for 64 bit stack bias
	#endif
	  __ sub(Lmonitors, BytesPerWord, Lesp);       // set Lesp
	
	  // setup interpreter activation registers
	  __ sub(Gargs, BytesPerWord, Llocals);        // set Llocals
	
  {- -------------------------------------------
  (1) コード生成: (ただし, ProfileInterpreter オプションが指定されていない場合は, 不要なので生成しない)
      「InterpreterMacroAssembler::set_method_data_pointer() が生成するコードにより
        ImethodDataPtr レジスタの値をセットしておく.
  
        (もし methodDataOop が生成されていれば, 
         DataLayout へのポインタがセットされる.
         まだ methodDataOop が生成されていないメソッドの場合には, 
         ImethodDataPtr の値は NULL になる)                 」
      ---------------------------------------- -}

	  if (ProfileInterpreter) {
	#ifdef FAST_DISPATCH
	    // FAST_DISPATCH and ProfileInterpreter are mutually exclusive since
	    // they both use I2.
	    assert(0, "FAST_DISPATCH and +ProfileInterpreter are mutually exclusive");
	#endif // FAST_DISPATCH
	    __ set_method_data_pointer();
	  }
	
  {- -------------------------------------------
  (1) (上記のコメントには, 最後に「必要があれば, non-argument locals に対して初期値を設定しておく」と書かれていたが, 
       ここにはそういったコードはない模様)
      ---------------------------------------- -}

	}
	
```


