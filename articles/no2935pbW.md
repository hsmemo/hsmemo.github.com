---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vframeArray.cpp

### 名前(function name)
```
void vframeArrayElement::unpack_on_stack(int caller_actual_parameters,
                                         int callee_parameters,
                                         int callee_locals,
                                         frame* caller,
                                         bool is_top_frame,
                                         int exec_mode) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* thread = (JavaThread*) Thread::current();
	
	  // Look at bci and decide on bcp and continuation pc
	  address bcp;
	  // C++ interpreter doesn't need a pc since it will figure out what to do when it
	  // begins execution
	  address pc;
	  bool use_next_mdp = false; // true if we should use the mdp associated with the next bci
	                             // rather than the one associated with bcp

  {- -------------------------------------------
  (1) 返り先を決定して pc という変数に格納する
  
      *
  
      * should_reexecute() なら,
        返り先は Interpreter::deopt_reexecute_entry()
        (AbstractInterpreter::deopt_reexecute_entry() は C1 の時は特殊な処理があるが, 
         そうでなければ Interpreter::deopt_entry() を呼ぶだけ.)
  
      * それ以外の場合なら,
        返り先は Interpreter::deopt_continue_after_entry()
        (AbstractInterpreter::deopt_continue_after_entry() は
         Interpreter::deopt_entry() か Interpreter::return_entry() を呼ぶ.
         どちらを呼ぶかは top_frame かどうか(= is_top_frame 引数が true かどうか)で決まる.)
  
      #TODO 具体的にどういうケースではどこが返り先になる?
      ---------------------------------------- -}

	  if (raw_bci() == SynchronizationEntryBCI) {
	    // We are deoptimizing while hanging in prologue code for synchronized method
	    bcp = method()->bcp_from(0); // first byte code
	    pc  = Interpreter::deopt_entry(vtos, 0); // step = 0 since we don't skip current bytecode
	  } else if (should_reexecute()) { //reexecute this bytecode
	    assert(is_top_frame, "reexecute allowed only for the top frame");
	    bcp = method()->bcp_from(bci());
	    pc  = Interpreter::deopt_reexecute_entry(method(), bcp);
	  } else {
	    bcp = method()->bcp_from(bci());
	    pc  = Interpreter::deopt_continue_after_entry(method(), bcp, callee_parameters, is_top_frame);
	    use_next_mdp = true;
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Bytecodes::is_defined(*bcp), "must be a valid bytecode");
	
  {- -------------------------------------------
  (1) (assert)
      
      (monitorenter との関係については, ... #TODO)
      ---------------------------------------- -}

	  // Monitorenter and pending exceptions:
	  //
	  // For Compiler2, there should be no pending exception when deoptimizing at monitorenter
	  // because there is no safepoint at the null pointer check (it is either handled explicitly
	  // or prior to the monitorenter) and asynchronous exceptions are not made "pending" by the
	  // runtime interface for the slow case (see JRT_ENTRY_FOR_MONITORENTER).  If an asynchronous
	  // exception was processed, the bytecode pointer would have to be extended one bytecode beyond
	  // the monitorenter to place it in the proper exception range.
	  //
	  // For Compiler1, deoptimization can occur while throwing a NullPointerException at monitorenter,
	  // in which case bcp should point to the monitorenter since it is within the exception's range.
	
	  assert(*bcp != Bytecodes::_monitorenter || is_top_frame, "a _monitorenter must be a top frame");
	  assert(thread->deopt_nmethod() != NULL, "nmethod should be known");
	  guarantee(!(thread->deopt_nmethod()->is_compiled_by_c2() &&
	              *bcp == Bytecodes::_monitorenter             &&
	              exec_mode == Deoptimization::Unpack_exception),
	            "shouldn't get exception during monitorenter");
	
  {- -------------------------------------------
  (1) top_frame の場合(= is_top_frame 引数が true の場合)は, 
      特殊な返り先が必要なケースが有るのでここで調整する.
  
      * JVMTI の PopFrame() でポップされたフレームの場合:
        返り先は, #ifndef CC_INTERP の場合は Interpreter::remove_activation_preserving_args_entry()  
        (<= PopFrame 処理のエントリポイント)
  
        (#ifndef CC_INTERP でない場合は Interpreter::deopt_entry() になっているが... #TODO)
  
      * JVMTI の PopFrame() でポップされたフレームの呼び出し元のフレームである場合:
        返り先は Interpreter::deopt_entry(vtos, 0)                                                   
        (<= 0 なので invoke 命令を再実行する処理になる)
  
      * JVMTI の ForceEarlyReturn*() の処理を行っている場合:
        返り先は, #ifndef CC_INTERP の場合は Interpreter::remove_activation_early_entry()
        (<= ForceEarlyReturn* 処理のエントリポイント)
  
        (なお, #ifndef CC_INTERP でない場合は実装されてないようだが... #TODO)
  
      * それ以外の場合:
        *
        * pending exception がある場合 (Deoptimization::Unpack_exception)
          返り先は SharedRuntime::raw_exception_handler_for_return_address()
        * uncommon trap の場合 (Deoptimization::Unpack_uncommon_trap)
          返り先は Interpreter::deopt_entry(vtos, 0)                                                  
          (<= 0 なので invoke 命令を再実行する処理になる)
        *
  
      ---------------------------------------- -}

	  int popframe_preserved_args_size_in_bytes = 0;
	  int popframe_preserved_args_size_in_words = 0;
	  if (is_top_frame) {

    {- -------------------------------------------
  (1.1) (JVMTI の PopFrame() の処理を行っている場合)
        ---------------------------------------- -}

	    JvmtiThreadState *state = thread->jvmti_thread_state();
	    if (JvmtiExport::can_pop_frame() &&
	        (thread->has_pending_popframe() || thread->popframe_forcing_deopt_reexecution())) {

      {- -------------------------------------------
  (1.1.1) (JVMTI の PopFrame() でポップされたフレームである場合)
          ---------------------------------------- -}

	      if (thread->has_pending_popframe()) {
	        // Pop top frame after deoptimization
	#ifndef CC_INTERP
	        pc = Interpreter::remove_activation_preserving_args_entry();
	#else
	        // Do an uncommon trap type entry. c++ interpreter will know
	        // to pop frame and preserve the args
	        pc = Interpreter::deopt_entry(vtos, 0);
	        use_next_mdp = false;
	#endif

      {- -------------------------------------------
  (1.1.1) (JVMTI の PopFrame() でポップされたフレームの親フレームである場合)
          ---------------------------------------- -}

	      } else {
	        // Reexecute invoke in top frame
	        pc = Interpreter::deopt_entry(vtos, 0);
	        use_next_mdp = false;
	        popframe_preserved_args_size_in_bytes = in_bytes(thread->popframe_preserved_args_size());
	        // Note: the PopFrame-related extension of the expression stack size is done in
	        // Deoptimization::fetch_unroll_info_helper
	        popframe_preserved_args_size_in_words = in_words(thread->popframe_preserved_args_size_in_words());
	      }

    {- -------------------------------------------
  (1.1) (JVMTI の ForceEarlyReturn*() の処理を行っている場合)
        ---------------------------------------- -}

	    } else if (JvmtiExport::can_force_early_return() && state != NULL && state->is_earlyret_pending()) {
	      // Force early return from top frame after deoptimization
	#ifndef CC_INTERP
	      pc = Interpreter::remove_activation_early_entry(state->earlyret_tos());
	#else
	     // TBD: Need to implement ForceEarlyReturn for CC_INTERP (ia64)
	#endif

    {- -------------------------------------------
  (1.1) (それ以外の場合)
        ---------------------------------------- -}

	    } else {
	      // Possibly override the previous pc computation of the top (youngest) frame
	      switch (exec_mode) {
	      case Deoptimization::Unpack_deopt:
	        // use what we've got
	        break;
	      case Deoptimization::Unpack_exception:
	        // exception is pending
	        pc = SharedRuntime::raw_exception_handler_for_return_address(thread, pc);
	        // [phh] We're going to end up in some handler or other, so it doesn't
	        // matter what mdp we point to.  See exception_handler_for_exception()
	        // in interpreterRuntime.cpp.
	        break;
	      case Deoptimization::Unpack_uncommon_trap:
	      case Deoptimization::Unpack_reexecute:
	        // redo last byte code
	        pc  = Interpreter::deopt_entry(vtos, 0);
	        use_next_mdp = false;
	        break;
	      default:
	        ShouldNotReachHere();
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) AbstractInterpreter::layout_activation() を呼んで, フレーム内に値を設定する.
      (ここで行う処理は, メソッドのエントリ部で行われている処理に相当.
      具体的には, sparc 版なら InterpreterGenerator::generate_fixed_frame(),
      x86_64 なら AbstractInterpreterGenerator::generate_method_entry() で行われている処理.)
      ---------------------------------------- -}

	  // Setup the interpreter frame
	
	  assert(method() != NULL, "method must exist");
	  int temps = expressions()->size();
	
	  int locks = monitors() == NULL ? 0 : monitors()->number_of_monitors();
	
	  Interpreter::layout_activation(method(),
	                                 temps + callee_parameters,
	                                 popframe_preserved_args_size_in_words,
	                                 locks,
	                                 caller_actual_parameters,
	                                 callee_parameters,
	                                 callee_locals,
	                                 caller,
	                                 iframe(),
	                                 is_top_frame);
	
  {- -------------------------------------------
  (1) frame 内のリターンアドレスを (pc に格納した返り先へと) 書き換える  
      (このリターンアドレスは, 最後に restore/return した際に使用される(= ここにジャンプする)).
      ---------------------------------------- -}

	  // Update the pc in the frame object and overwrite the temporary pc
	  // we placed in the skeletal frame now that we finally know the
	  // exact interpreter address we should use.
	
	  _frame.patch_pc(thread, pc);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (!method()->is_synchronized() || locks > 0, "synchronized methods must have monitors");
	
  {- -------------------------------------------
  (1) フレーム中の monitor の領域 (BasicObjectLock 領域)に値を設定する.
      (正しい? メモリ領域に書き込んでいる感じがあまりしないが... #TODO)
      ---------------------------------------- -}

	  BasicObjectLock* top = iframe()->interpreter_frame_monitor_begin();
	  for (int index = 0; index < locks; index++) {
	    top = iframe()->previous_monitor_in_interpreter_frame(top);
	    BasicObjectLock* src = _monitors->at(index);
	    top->set_obj(src->obj());
	    src->lock()->move_to(src->obj(), top->lock());
	  }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (ProfileInterpreter) {
	    iframe()->interpreter_frame_set_mdx(0); // clear out the mdp.
	  }

  {- -------------------------------------------
  (1) 次の実行するバイトコードをセットしておく. 
      (で正しいか??.. ところでこれは後でどう使われる?) #TODO
      ---------------------------------------- -}

	  iframe()->interpreter_frame_set_bcx((intptr_t)bcp); // cannot use bcp because frame is not initialized yet

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (ProfileInterpreter) {
	    methodDataOop mdo = method()->method_data();
	    if (mdo != NULL) {
	      int bci = iframe()->interpreter_frame_bci();
	      if (use_next_mdp) ++bci;
	      address mdp = mdo->bci_to_dp(bci);
	      iframe()->interpreter_frame_set_mdp(mdp);
	    }
	  }
	
  {- -------------------------------------------
  (1) フレーム中のオペランドスタック (expression stack) に値を設定する
      (オペランドスタック内の要素の個数分だけループして, 対応する値をメモリに書き込んでいく)
      (なお top frame でない場合は, オペランドスタックの要素のうちメソッド呼び出しの引数に当たる分は展開しない.
       引数については, 代わりに callee 側の局所変数領域に展開される.)
      ---------------------------------------- -}

	  // Unpack expression stack
	  // If this is an intermediate frame (i.e. not top frame) then this
	  // only unpacks the part of the expression stack not used by callee
	  // as parameters. The callee parameters are unpacked as part of the
	  // callee locals.
	  int i;
	  for(i = 0; i < expressions()->size(); i++) {
	    StackValue *value = expressions()->at(i);
	    intptr_t*   addr  = iframe()->interpreter_frame_expression_stack_at(i);
	    switch(value->type()) {
	      case T_INT:
	        *addr = value->get_int();
	        break;
	      case T_OBJECT:
	        *addr = value->get_int(T_OBJECT);
	        break;
	      case T_CONFLICT:
	        // A dead stack slot.  Initialize to null in case it is an oop.
	        *addr = NULL_WORD;
	        break;
	      default:
	        ShouldNotReachHere();
	    }
	  }
	
	
  {- -------------------------------------------
  (1) フレーム中の局所変数領域に値を設定する
      (局所変数領域の大きさ分だけループして, 対応する値をメモリに書き込んでいく)
      ---------------------------------------- -}

	  // Unpack the locals
	  for(i = 0; i < locals()->size(); i++) {
	    StackValue *value = locals()->at(i);
	    intptr_t* addr  = iframe()->interpreter_frame_local_at(i);
	    switch(value->type()) {
	      case T_INT:
	        *addr = value->get_int();
	        break;
	      case T_OBJECT:
	        *addr = value->get_int(T_OBJECT);
	        break;
	      case T_CONFLICT:
	        // A dead location. If it is an oop then we need a NULL to prevent GC from following it
	        *addr = NULL_WORD;
	        break;
	      default:
	        ShouldNotReachHere();
	    }
	  }
	
  {- -------------------------------------------
  (1) もし JVMTI の PopFrame() によってこの frame から呼び出していたメソッドがポップされていた場合, 以下の処理を行う.
      (ポップされたメソッドを再度呼び出すために, PopFrame 処理が退避していた引数をオペランドスタックにセットする処理.
       (See: Interpreter::remove_activation_preserving_args_entry()))
      ---------------------------------------- -}

	  if (is_top_frame && JvmtiExport::can_pop_frame() && thread->popframe_forcing_deopt_reexecution()) {
	    // An interpreted frame was popped but it returns to a deoptimized
	    // frame. The incoming arguments to the interpreted activation
	    // were preserved in thread-local storage by the
	    // remove_activation_preserving_args_entry in the interpreter; now
	    // we put them back into the just-unpacked interpreter frame.
	    // Note that this assumes that the locals arena grows toward lower
	    // addresses.
	    if (popframe_preserved_args_size_in_words != 0) {
	      void* saved_args = thread->popframe_preserved_args();
	      assert(saved_args != NULL, "must have been saved by interpreter");
	#ifdef ASSERT
	      assert(popframe_preserved_args_size_in_words <=
	             iframe()->interpreter_frame_expression_stack_size()*Interpreter::stackElementWords,
	             "expression stack size should have been extended");
	#endif // ASSERT
	      int top_element = iframe()->interpreter_frame_expression_stack_size()-1;
	      intptr_t* base;
	      if (frame::interpreter_frame_expression_stack_direction() < 0) {
	        base = iframe()->interpreter_frame_expression_stack_at(top_element);
	      } else {
	        base = iframe()->interpreter_frame_expression_stack();
	      }
	      Copy::conjoint_jbytes(saved_args,
	                            base,
	                            popframe_preserved_args_size_in_bytes);
	      thread->popframe_free_preserved_args();
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (TraceDeoptimization && Verbose) {
	    ttyLocker ttyl;
	    tty->print_cr("[%d Interpreted Frame]", ++unpack_counter);
	    iframe()->print_on(tty);
	    RegisterMap map(thread);
	    vframe* f = vframe::new_vframe(iframe(), &map, thread);
	    f->print();
	
	    tty->print_cr("locals size     %d", locals()->size());
	    tty->print_cr("expression size %d", expressions()->size());
	
	    method()->print_value();
	    tty->cr();
	    // method()->print_codes();
	  } else if (TraceDeoptimization) {
	    tty->print("     ");
	    method()->print_value();
	    Bytecodes::Code code = Bytecodes::java_code_at(method(), bcp);
	    int bci = method()->bci_from(bcp);
	    tty->print(" - %s", Bytecodes::name(code));
	    tty->print(" @ bci %d ", bci);
	    tty->print_cr("sp = " PTR_FORMAT, iframe()->sp());
	  }
	#endif // PRODUCT
	
  {- -------------------------------------------
  (1) ??(不要な dangling pointer を残さないよう, このオブジェクト内のフィールドを NULL 化している模様)
      ---------------------------------------- -}

	  // The expression stack and locals are in the resource area don't leave
	  // a dangling pointer in the vframeArray we leave around for debug
	  // purposes
	
	  _locals = _expressions = NULL;
	
	}
	
```


