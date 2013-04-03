---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp

### 名前(function name)
```
  address generate_call_stub(address& return_address) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert((int)frame::entry_frame_after_call_words == -(int)rsp_after_call_off + 1 &&
	           (int)frame::entry_frame_call_wrapper_offset == (int)call_wrapper_off,
	           "adjust this code");

  {- -------------------------------------------
  (1) (以下, StubCodeMark によるスタブ生成を行う) (See: StubCodeMark)
      ---------------------------------------- -}

	    StubCodeMark mark(this, "StubRoutines", "call_stub");

  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	    address start = __ pc();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (call_wrapper から thread までが, スタブコードに渡された引数の待避先)
      (r15_save から rbx_save までが, callee save レジスタの待避先)
      ---------------------------------------- -}

	    // same as in generate_catch_exception()!
	    const Address rsp_after_call(rbp, rsp_after_call_off * wordSize);
	
	    const Address call_wrapper  (rbp, call_wrapper_off   * wordSize);
	    const Address result        (rbp, result_off         * wordSize);
	    const Address result_type   (rbp, result_type_off    * wordSize);
	    const Address method        (rbp, method_off         * wordSize);
	    const Address entry_point   (rbp, entry_point_off    * wordSize);
	    const Address parameters    (rbp, parameters_off     * wordSize);
	    const Address parameter_size(rbp, parameter_size_off * wordSize);
	
	    // same as in generate_catch_exception()!
	    const Address thread        (rbp, thread_off         * wordSize);
	
	    const Address r15_save(rbp, r15_off * wordSize);
	    const Address r14_save(rbp, r14_off * wordSize);
	    const Address r13_save(rbp, r13_off * wordSize);
	    const Address r12_save(rbp, r12_off * wordSize);
	    const Address rbx_save(rbp, rbx_off * wordSize);
	
  {- -------------------------------------------
  (1) (以下で, スタブコードを生成する)
      ---------------------------------------- -}

	    // stub code

  {- -------------------------------------------
  (1) コード生成:
      「rbp をスタック上に待避し, rbp の値を rsp の値に変更」
      ---------------------------------------- -}

	    __ enter();

  {- -------------------------------------------
  (1) コード生成:
      「引数が入っているレジスタや callee save レジスタを待避するスペースが必要なので, 
        rsp を rsp_after_call_off 分だけずらしておく」
      ---------------------------------------- -}

	    __ subptr(rsp, -rsp_after_call_off * wordSize);
	
  {- -------------------------------------------
  (1) コード生成:
      「レジスタに入っている引数の値をスタック上に待避しておく」
      ---------------------------------------- -}

	    // save register parameters
	#ifndef _WIN64
	    __ movptr(parameters,   c_rarg5); // parameters
	    __ movptr(entry_point,  c_rarg4); // entry_point
	#endif
	
	    __ movptr(method,       c_rarg3); // method
	    __ movl(result_type,  c_rarg2);   // result type
	    __ movptr(result,       c_rarg1); // result
	    __ movptr(call_wrapper, c_rarg0); // call wrapper
	
  {- -------------------------------------------
  (1) コード生成:
      「callee save レジスタをスタック上に待避しておく」
      ---------------------------------------- -}

	    // save regs belonging to calling function
	    __ movptr(rbx_save, rbx);
	    __ movptr(r12_save, r12);
	    __ movptr(r13_save, r13);
	    __ movptr(r14_save, r14);
	    __ movptr(r15_save, r15);
	#ifdef _WIN64
	    for (int i = 6; i <= 15; i++) {
	      __ movdqu(xmm_save(i), as_XMMRegister(i));
	    }
	
	    const Address rdi_save(rbp, rdi_off * wordSize);
	    const Address rsi_save(rbp, rsi_off * wordSize);
	
	    __ movptr(rsi_save, rsi);
	    __ movptr(rdi_save, rdi);
	#else
	    const Address mxcsr_save(rbp, mxcsr_off * wordSize);
	    {
	      Label skip_ldmx;
	      __ stmxcsr(mxcsr_save);
	      __ movl(rax, mxcsr_save);
	      __ andl(rax, MXCSR_MASK);    // Only check control and mask bits
	      ExternalAddress mxcsr_std(StubRoutines::x86::mxcsr_std());
	      __ cmp32(rax, mxcsr_std);
	      __ jcc(Assembler::equal, skip_ldmx);
	      __ ldmxcsr(mxcsr_std);
	      __ bind(skip_ldmx);
	    }
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「これから Java コードの実行が始まるので, G2_thread レジスタの値を適切に設定しておく.
        (カレントスレッドに対応する JavaThread のアドレスが
         第8引数として渡されているはずなので, それをロード)」
      ---------------------------------------- -}

	    // Load up thread register
	    __ movptr(r15_thread, thread);

  {- -------------------------------------------
  (1) コード生成:
      「G6_heapbase レジスタに正しい値をセットしておく」
      (See: [here](no289165bb.html) for details)
      ---------------------------------------- -}

	    __ reinit_heapbase();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    // make sure we have no pending exceptions
	    {
	      Label L;
	      __ cmpptr(Address(r15_thread, Thread::pending_exception_offset()), (int32_t)NULL_WORD);
	      __ jcc(Assembler::equal, L);
	      __ stop("StubRoutines::call_stub: entered with pending exception");
	      __ bind(L);
	    }
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出す Java メソッドに引数がある場合は, スタック上にコピーしておく.
        (これが, 呼び出した Java メソッドの局所変数領域(の一部)になる)」
  
      (スタブコードが呼ばれた時点では, Java メソッド用の引数は JavaCallArguments オブジェクト内に格納されている.
       そして, スタブコードの第6引数がその JavaCallArguments オブジェクトを指している.
       局所変数領域は現在の SP の直下になるので, もし引数がある場合にはその位置にプッシュしていく)
      ---------------------------------------- -}

	    // pass parameters if any
	    BLOCK_COMMENT("pass parameters if any");
	    Label parameters_done;

    {- -------------------------------------------
  (1.1) (呼び出す Java メソッドの引数の個数が 0 ならば何もしない (parameters_done ラベルにジャンプ).
         そうでなければループに突入.)
        ---------------------------------------- -}

	    __ movl(c_rarg3, parameter_size);
	    __ testl(c_rarg3, c_rarg3);
	    __ jcc(Assembler::zero, parameters_done);
	
	    Label loop;
	    __ movptr(c_rarg2, parameters);       // parameter pointer
	    __ movl(c_rarg1, c_rarg3);            // parameter counter is in c_rarg1

    {- -------------------------------------------
  (1.1) (ここからがループの始まり. 全ての引数をスタック上にプッシュしていく)
        ---------------------------------------- -}

	    __ BIND(loop);
	    __ movptr(rax, Address(c_rarg2, 0));// get parameter
	    __ addptr(c_rarg2, wordSize);       // advance to next parameter
	    __ decrementl(c_rarg1);             // decrement counter
	    __ push(rax);                       // pass parameter
	    __ jcc(Assembler::notZero, loop);

    {- -------------------------------------------
  (1.1) (ここまでがループ)
        ---------------------------------------- -}

  {- -------------------------------------------
  (1) (ここまでが引数をスタック上にコピーする処理)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) (以下で, 実際に Java メソッドを呼び出す)
      ---------------------------------------- -}

	    // call Java function
	    __ BIND(parameters_done);

  {- -------------------------------------------
  (1) コード生成:
      「以下のレジスタに値をセットした後, 
        call 命令により, 第5引数で指定されたエントリポイントを呼び出す.
        * rbx 
          第4引数で渡された methodOop のアドレスをセット
        * r13
          現在の rsp の値をセット (<= セットというか待避しておく処理) 」
      ---------------------------------------- -}

	    __ movptr(rbx, method);             // get methodOop
	    __ movptr(c_rarg1, entry_point);    // get entry_point
	    __ mov(r13, rsp);                   // set sender sp
	    BLOCK_COMMENT("call Java function");
	    __ call(c_rarg1);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (See: BLOCK_COMMENT)
      ---------------------------------------- -}

	    BLOCK_COMMENT("call_stub_return_address:");

  {- -------------------------------------------
  (1) この時点でのコードのアドレスを取得し, 引数で渡された return_pc に記録しておく.
      (これは, 上の call による呼び出しのリターンアドレスに相当.
       return_pc は参照渡しであり, この値は呼び出し元から使用される)
      ---------------------------------------- -}

	    return_address = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出した Java メソッドの返値を, 第2引数で指定されたアドレスに書き込む.
  
        (なお, 返値の型に応じて適切なストア命令を実行する.
         返値の型に関する情報は第3引数で渡されてくるので, その情報を動的にチェックして適切なストア命令を選ぶ)」
      ---------------------------------------- -}

	    // store result depending on type (everything that is not
	    // T_OBJECT, T_LONG, T_FLOAT or T_DOUBLE is treated as T_INT)
	    __ movptr(c_rarg0, result);
	    Label is_long, is_float, is_double, exit;
	    __ movl(c_rarg1, result_type);
	    __ cmpl(c_rarg1, T_OBJECT);
	    __ jcc(Assembler::equal, is_long);
	    __ cmpl(c_rarg1, T_LONG);
	    __ jcc(Assembler::equal, is_long);
	    __ cmpl(c_rarg1, T_FLOAT);
	    __ jcc(Assembler::equal, is_float);
	    __ cmpl(c_rarg1, T_DOUBLE);
	    __ jcc(Assembler::equal, is_double);
	
    {- -------------------------------------------
  (1.1) (ここが T_INT だった場合の処理. 
         それ以外のケースの処理 (is_long ラベル, is_float ラベル, is_double ラベル) は
         この関数の末尾で生成されている)
        ---------------------------------------- -}

	    // handle T_INT case
	    __ movl(Address(c_rarg0, 0), rax);
	
	    __ BIND(exit);
	
  {- -------------------------------------------
  (1) コード生成:
      「(この時点では引数をプッシュした分だけ rsp の値が変わってしまっているので)
        rsp を引数をプッシュする前の値に戻しておく.」
      ---------------------------------------- -}

	    // pop parameters
	    __ lea(rsp, rsp_after_call);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    // verify that threads correspond
	    {
	      Label L, S;
	      __ cmpptr(r15_thread, thread);
	      __ jcc(Assembler::notEqual, S);
	      __ get_thread(rbx);
	      __ cmpptr(r15_thread, rbx);
	      __ jcc(Assembler::equal, L);
	      __ bind(S);
	      __ jcc(Assembler::equal, L);
	      __ stop("StubRoutines::call_stub: threads must correspond");
	      __ bind(L);
	    }
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「callee save レジスタの値を復帰させる」
      ---------------------------------------- -}

	    // restore regs belonging to calling function
	#ifdef _WIN64
	    for (int i = 15; i >= 6; i--) {
	      __ movdqu(as_XMMRegister(i), xmm_save(i));
	    }
	#endif
	    __ movptr(r15, r15_save);
	    __ movptr(r14, r14_save);
	    __ movptr(r13, r13_save);
	    __ movptr(r12, r12_save);
	    __ movptr(rbx, rbx_save);
	
	#ifdef _WIN64
	    __ movptr(rdi, rdi_save);
	    __ movptr(rsi, rsi_save);
	#else
	    __ ldmxcsr(mxcsr_save);
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「rsp の値を, スタブコードが呼び出された時点の値に戻す」
      ---------------------------------------- -}

	    // restore rsp
	    __ addptr(rsp, -rsp_after_call_off * wordSize);
	
  {- -------------------------------------------
  (1) コード生成:
      「rbp の値を元に戻し, リターンする」
      ---------------------------------------- -}

	    // return
	    __ pop(rbp);
	    __ ret(0);
	
  {- -------------------------------------------
  (1) コード生成:
      「(以下は, 返値が T_INT 以外の場合の処理)
        それぞれの型に応じた適切なストア命令を実行した後, exit ラベルに飛んで本筋の処理に合流する.」
      ---------------------------------------- -}

	    // handle return types different from T_INT
	    __ BIND(is_long);
	    __ movq(Address(c_rarg0, 0), rax);
	    __ jmp(exit);
	
	    __ BIND(is_float);
	    __ movflt(Address(c_rarg0, 0), xmm0);
	    __ jmp(exit);
	
	    __ BIND(is_double);
	    __ movdbl(Address(c_rarg0, 0), xmm0);
	    __ jmp(exit);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	    return start;
	  }
	
```


