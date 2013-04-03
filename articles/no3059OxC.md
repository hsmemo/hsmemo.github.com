---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp

### 名前(function name)
```
void TemplateInterpreterGenerator::generate_throw_exception() {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (ここからが, 例外が rethrow された場合の処理コード.
       ここで生成したコードは Interpreter::rethrow_exception_entry() からアクセスされる.)
  
      (とは言っても, 処理自体は例外送出時と同じなので, このまま以下の
       Interpreter::_throw_exception_entry の処理にフォールスルーするだけ)
      ---------------------------------------- -}

	  // Entry point in previous activation (i.e., if the caller was interpreted)
	  Interpreter::_rethrow_exception_entry = __ pc();
	  // O0: exception
	
  {- -------------------------------------------
  (1) (ここからが, 例外発生時の例外送出処理コード.
       ここで生成したコードは Interpreter::throw_exception_entry() からアクセスされる.)
      ---------------------------------------- -}

	  // entry point for exceptions thrown within interpreter code
	  Interpreter::_throw_exception_entry = __ pc();

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_thread();
	  // expression stack is undefined here
	  // O0: exception, i.e. Oexception
	  // Lbcp: exception bcx
	  __ verify_oop(Oexception);
	
	
  {- -------------------------------------------
  (1) コード生成:
      「オペランドスタック("expression stack")を空にする.」
      ---------------------------------------- -}

	  // expression stack must be empty before entering the VM in case of an exception
	  __ empty_expression_stack();

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterRuntime::exception_handler_for_exception() を呼んで
        対応する例外ハンドラのエントリポイント(対応するものがなければ unwind 処理のエントリポイント)を取得し, 
        そのアドレスにジャンプする.
      
        (なお, 例外ハンドラ内で例外オブジェクトにアクセスできるよう
         ジャンプ前に push_ptr() で例外オブジェクトをオペランドスタックに積んでいる)」
      ---------------------------------------- -}

	  // find exception handler address and preserve exception oop
	  // call C routine to find handler and jump to it
	  __ call_VM(O1, CAST_FROM_FN_PTR(address, InterpreterRuntime::exception_handler_for_exception), Oexception);
	  __ push_ptr(O1); // push exception for exception handler bytecodes
	
	  __ JMP(O0, 0); // jump to exception handler (may be remove activation entry!)
	  __ delayed()->nop();
	
	
	  // if the exception is not handled in the current frame
	  // the frame is removed and the exception is rethrown
	  // (i.e. exception continuation is _rethrow_exception)
	  //
	  // Note: At this point the bci is still the bxi for the instruction which caused
	  //       the exception and the expression stack is empty. Thus, for any VM calls
	  //       at this point, GC will find a legal oop map (with empty expression stack).
	
	  // in current activation
	  // tos: exception
	  // Lbcp: exception bcp

  {- -------------------------------------------
  (1) (ここまでが, 例外発生時の例外送出処理用のコード)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) (ここからは, JVMTI の PopFrame() 処理用のコード.
       ここで生成したコードは Interpreter::remove_activation_preserving_args_entry() からアクセスされる.)
      ---------------------------------------- -}

	  //
	  // JVMTI PopFrame support
	  //
	
	  Interpreter::_remove_activation_preserving_args_entry = __ pc();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Address popframe_condition_addr(G2_thread, JavaThread::popframe_condition_offset());

  {- -------------------------------------------
  (1) コード生成:
      「これ以降に起こった call_VM でまたこの関数に re-enter してしまわないように 
        (= PopFrame については既に作業中であることを示すために), 
        カレントスレッドの JavaThread::_popframe_condition フィールド中の 
        JavaThread::popframe_processing_bit ビットを立てておく.」
      ---------------------------------------- -}

	  // Set the popframe_processing bit in popframe_condition indicating that we are
	  // currently handling popframe, so that call_VMs that may happen later do not trigger new
	  // popframe handling cycles.
	
	  __ ld(popframe_condition_addr, G3_scratch);
	  __ or3(G3_scratch, JavaThread::popframe_processing_bit, G3_scratch);
	  __ stw(G3_scratch, popframe_condition_addr);
	
  {- -------------------------------------------
  (1) コード生成:
      「オペランドスタック("expression stack")を空にする.」
      (これは通常の例外処理等と同じ)
      ---------------------------------------- -}

	  // Empty the expression stack, as in normal exception handling
	  __ empty_expression_stack();

  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::unlock_if_synchronized_method() が生成するコードで, 
       (もし synchronized メソッドの場合には) ロックを解放する.
       モニターのロック状態がおかしければ, IllegalMonitorStateException も送出される」
      ---------------------------------------- -}

	  __ unlock_if_synchronized_method(vtos, /* throw_monitor_exception */ false, /* install_monitor_exception */ false);
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterRuntime::interpreter_contains() を呼んで
        PopFrame() 対象になっているフレームの呼び出し元を確認し, その種類に応じて処理を分岐.
        * 呼び出し元が compiled frame の場合:
          このままフォールスルー
        * 呼び出し元が interpreter frame の場合:
          caller_not_deoptimized ラベルまでジャンプ                             」
      ---------------------------------------- -}

	  {
	    // Check to see whether we are returning to a deoptimized frame.
	    // (The PopFrame call ensures that the caller of the popped frame is
	    // either interpreted or compiled and deoptimizes it if compiled.)
	    // In this case, we can't call dispatch_next() after the frame is
	    // popped, but instead must save the incoming arguments and restore
	    // them after deoptimization has occurred.
	    //
	    // Note that we don't compare the return PC against the
	    // deoptimization blob's unpack entry because of the presence of
	    // adapter frames in C2.
	    Label caller_not_deoptimized;
	    __ call_VM_leaf(L7_thread_cache, CAST_FROM_FN_PTR(address, InterpreterRuntime::interpreter_contains), I7);
	    __ tst(O0);
	    __ brx(Assembler::notEqual, false, Assembler::pt, caller_not_deoptimized);
	    __ delayed()->nop();
	
  {- -------------------------------------------
  (1) コード生成:
      「(以下が, 呼び出し元が compiled frame の場合の処理)
  
        この場合は, PopFrame() 対象のフレームが破棄された後, 
        呼び出し元に対する deopt 処理が行われ, それが終わった後で invoke 命令が再実行される.
  
        (引数については deopt 処理中に再セットが行われ, deopt 終了後には 
         bci が invoke 命令を指した状態で interpreter に引き継がれる.
         See: vframeArrayElement::unpack_on_stack())                          」
      ---------------------------------------- -}

	    const Register Gtmp1 = G3_scratch;
	    const Register Gtmp2 = G1_scratch;
	
    {- -------------------------------------------
  (1.1) コード生成:
        「invoke を再実行するために必要な引数の情報を
          Deoptimization::popframe_preserve_args() で退避する.」
        ---------------------------------------- -}

	    // Compute size of arguments for saving when returning to deoptimized caller
	    __ lduh(Lmethod, in_bytes(methodOopDesc::size_of_parameters_offset()), Gtmp1);
	    __ sll(Gtmp1, Interpreter::logStackElementSize, Gtmp1);
	    __ sub(Llocals, Gtmp1, Gtmp2);
	    __ add(Gtmp2, wordSize, Gtmp2);
	    // Save these arguments
	    __ call_VM_leaf(L7_thread_cache, CAST_FROM_FN_PTR(address, Deoptimization::popframe_preserve_args), G2_thread, Gtmp1, Gtmp2);

    {- -------------------------------------------
  (1.1) コード生成:
        「deopt 処理中で, 今回の処理対象が「PopFrame() 対象の呼び出し元」であることが把握できるよう, 
          JavaThread::_popframe_condition フィールドの 
          JavaThread::popframe_force_deopt_reexecution_bit ビットが立てておく.」
        ---------------------------------------- -}

	    // Inform deoptimization that it is responsible for restoring these arguments
	    __ set(JavaThread::popframe_force_deopt_reexecution_bit, Gtmp1);
	    Address popframe_condition_addr(G2_thread, JavaThread::popframe_condition_offset());
	    __ st(Gtmp1, popframe_condition_addr);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「restore 命令でレジスタを復帰(SP も I5_savedSP に復帰)させつつ, リターンする.
          (restore で PopFrame() 対象になったフレームを破棄し, 
          リターンすることで呼び出し元に対する deopt 処理を開始させる)」
        ---------------------------------------- -}

	    // Return from the current method
	    // The caller's SP was adjusted upon method entry to accomodate
	    // the callee's non-argument locals. Undo that adjustment.
	    __ ret();
	    __ delayed()->restore(I5_savedSP, G0, SP);
	
  {- -------------------------------------------
  (1) コード生成:
      「(以下が, 呼び出し元が interpreter frame の場合の処理)」
      ---------------------------------------- -}

	    __ bind(caller_not_deoptimized);
	  }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「カレントスレッドの JavaThread::_popframe_condition フィールドをリセットする.」
        ---------------------------------------- -}

	  // Clear the popframe condition flag
	  __ stw(G0 /* popframe_inactive */, popframe_condition_addr);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「restore 命令で PopFrame() 対象のフレームを破棄し
          (ついでにレジスタを復帰させ, SP も I5_savedSP に復帰させ), 
          dispatch_next() で再度 invoke 命令を実行させる.
          (なお, ProfileInterpreter オプションが指定されていれば,
          既にインクリメントされてしまっている method data pointer を元に戻しておく) 」
        ---------------------------------------- -}

	  // Get out of the current method (how this is done depends on the particular compiler calling
	  // convention that the interpreter currently follows)
	  // The caller's SP was adjusted upon method entry to accomodate
	  // the callee's non-argument locals. Undo that adjustment.
	  __ restore(I5_savedSP, G0, SP);
	  // The method data pointer was incremented already during
	  // call profiling. We have to restore the mdp for the current bcp.
	  if (ProfileInterpreter) {
	    __ set_method_data_pointer_for_bcp();
	  }
	  // Resume bytecode interpretation at the current bcp
	  __ dispatch_next(vtos);

  {- -------------------------------------------
  (1) (ここまでが, JVMTI の PopFrame() 処理用のコード)
      ---------------------------------------- -}

	  // end of JVMTI PopFrame support
	
  {- -------------------------------------------
  (1) (ここからは, 例外発生時の unwind 処理用のコード.
      ここで生成したコードは Interpreter::remove_activation_entry() からアクセスされる.)
      ---------------------------------------- -}

	  Interpreter::_remove_activation_entry = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「送出された例外オブジェクトを Oexception レジスタに入れておく」
      ---------------------------------------- -}

	  // preserve exception over this code sequence (remove activation calls the vm, but oopmaps are not correct here)
	  __ pop_ptr(Oexception);                                  // get exception
	
  {- -------------------------------------------
  (1) コード生成:
      「InterpreterMacroAssembler::unlock_if_synchronized_method() が生成するコードで, 
       (もし synchronized メソッドの場合には) ロックを解放する.
       モニターのロック状態がおかしければ, IllegalMonitorStateException も送出される.
  
       (なお処理の前後で, set_vm_result()/get_vm_result() により
        送出された例外オブジェクトの退避復帰も行っている)           」
      ---------------------------------------- -}

	  // Intel has the following comment:
	  //// remove the activation (without doing throws on illegalMonitorExceptions)
	  // They remove the activation without checking for bad monitor state.
	  // %%% We should make sure this is the right semantics before implementing.
	
	  // %%% changed set_vm_result_2 to set_vm_result and get_vm_result_2 to get_vm_result. Is there a bug here?
	  __ set_vm_result(Oexception);
	  __ unlock_if_synchronized_method(vtos, /* throw_monitor_exception */ false);
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点) (DTrace のフック点)
      ---------------------------------------- -}

	  __ notify_method_exit(false, vtos, InterpreterMacroAssembler::SkipNotifyJVMTI);
	
	  __ get_vm_result(Oexception);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(Oexception);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    const int return_reg_adjustment = frame::pc_return_offset;
	  Address issuing_pc_addr(I7, return_reg_adjustment);
	
  {- -------------------------------------------
  (1) コード生成:
      「SharedRuntime::exception_handler_for_return_address() を呼んで, 
        呼び出し元のメソッドに応じた unwind 先アドレスを取得する. 
      
        (なお, レジスタについては, 以下のようにしている.
         * I* レジスタ, L* レジスタ
           値を変更しないようにしている ("caller's register window" のまま)
         * Oexception, Oissuing_pc
           unwind 先の例外ハンドラが必要とするため, 値をセットしている.
           なお, unwind 先には restore 後にジャンプするので, 
           セット対象のレジスタは restore を考慮して after_save() で取得している.)  」
      ---------------------------------------- -}

	  // We are done with this activation frame; find out where to go next.
	  // The continuation point will be an exception handler, which expects
	  // the following registers set up:
	  //
	  // Oexception: exception
	  // Oissuing_pc: the local call that threw exception
	  // Other On: garbage
	  // In/Ln:  the contents of the caller's register window
	  //
	  // We do the required restore at the last possible moment, because we
	  // need to preserve some state across a runtime call.
	  // (Remember that the caller activation is unknown--it might not be
	  // interpreted, so things like Lscratch are useless in the caller.)
	
	  // Although the Intel version uses call_C, we can use the more
	  // compact call_VM.  (The only real difference on SPARC is a
	  // harmlessly ignored [re]set_last_Java_frame, compared with
	  // the Intel code which lacks this.)
	  __ mov(Oexception,      Oexception ->after_save());  // get exception in I0 so it will be on O0 after restore
	  __ add(issuing_pc_addr, Oissuing_pc->after_save());  // likewise set I1 to a value local to the caller
	  __ super_call_VM_leaf(L7_thread_cache,
	                        CAST_FROM_FN_PTR(address, SharedRuntime::exception_handler_for_return_address),
	                        G2_thread, Oissuing_pc->after_save());
	
  {- -------------------------------------------
  (1) コード生成:
      「restore で現在のフレームが破棄しつつ, 
       (ついでにレジスタを復帰させ, SP も I5_savedSP に復帰させつつ)
       取得した unwind 先にジャンプする」
      ---------------------------------------- -}

	  // The caller's SP was adjusted upon method entry to accomodate
	  // the callee's non-argument locals. Undo that adjustment.
	  __ JMP(O0, 0);                         // return exception handler in caller
	  __ delayed()->restore(I5_savedSP, G0, SP);
	
	  // (same old exception object is already in Oexception; see above)
	  // Note that an "issuing PC" is actually the next PC after the call

  {- -------------------------------------------
  (1) (ここまでが, 例外発生時の unwind 処理用のコード)
      ---------------------------------------- -}

	}
	
```


