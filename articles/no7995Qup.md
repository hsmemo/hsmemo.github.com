---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/sharedRuntime_x86_64.cpp
### 説明(description)

```
// ---------------------------------------------------------------------------
// Generate a native wrapper for a given method.  The method takes arguments
// in the Java compiled code convention, marshals them to the native
// convention (handlizes oops, etc), transitions to native, makes the call,
// returns to java state (possibly blocking), unhandlizes any result and
// returns.
```

### 名前(function name)
```
nmethod *SharedRuntime::generate_native_wrapper(MacroAssembler *masm,
                                                methodHandle method,
                                                int compile_id,
                                                int total_in_args,
                                                int comp_args_on_stack,
                                                BasicType *in_sig_bt,
                                                VMRegPair *in_regs,
                                                BasicType ret_type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 生成するコード用の oopmap を用意しておく.
      ---------------------------------------- -}

	  // Native nmethod wrappers never take possesion of the oop arguments.
	  // So the caller will gc the arguments. The only thing we need an
	  // oopMap for is if the call is static
	  //
	  // An OopMap for lock (and class if static)
	  OopMapSet *oop_maps = new OopMapSet();

  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      (ここが "unverified entry point"(uep) の先頭)
      ---------------------------------------- -}

	  intptr_t start = (intptr_t)__ pc();
	
  {- -------------------------------------------
  (1) SharedRuntime::c_calling_convention() を呼んで, 
      ネイティブメソッド用の calling convention 及びレジスタに乗らない引数の個数を算出しておく.
      ---------------------------------------- -}

	  // We have received a description of where all the java arg are located
	  // on entry to the wrapper. We need to convert these args to where
	  // the jni function will expect them. To figure out where they go
	  // we convert the java signature to a C signature by inserting
	  // the hidden arguments as arg[0] and possibly arg[1] (static method)
	
	  int total_c_args = total_in_args + 1;
	  if (method->is_static()) {
	    total_c_args++;
	  }
	
	  BasicType* out_sig_bt = NEW_RESOURCE_ARRAY(BasicType, total_c_args);
	  VMRegPair* out_regs   = NEW_RESOURCE_ARRAY(VMRegPair,   total_c_args);
	
	  int argc = 0;
	  out_sig_bt[argc++] = T_ADDRESS;
	  if (method->is_static()) {
	    out_sig_bt[argc++] = T_OBJECT;
	  }
	
	  for (int i = 0; i < total_in_args ; i++ ) {
	    out_sig_bt[argc++] = in_sig_bt[i];
	  }
	
	  // Now figure out where the args must be stored and how much stack space
	  // they require.
	  //
	  int out_arg_slots;
	  out_arg_slots = c_calling_convention(out_sig_bt, out_regs, total_c_args);
	
  {- -------------------------------------------
  (1) (変数宣言など)
  
      (ここでは, JIT 生成するスタブ部分が必要とする「スタックフレームサイズ」を計算している.
       * synchronized の場合には, BasicObjectLock 用のスペースも忘れずに確保している
  
       * レジスタ渡しになっている oop については, ここで Handle 経由にしておかないと分からなくなってしまうので,
         フレーム内にはその Handle を確保するための領域も用意している. 
         ("// Now the space for the inbound oop handle area" 以下の行を参照)
  
         x86_64 なのでレジスタ渡しになるのは最大 6 個.
         現在の実装では, 実際に oop 引数がいくらあるかに関らず, 常時 6 個分の Handle スペースを確保しており, 
         対応が簡単に付くようにしている模様.
         そして, 後で呼ばれる object_move() の中で, Handle を作ってポインタ値を Handle 内に格納し, 
         代わりに引数はポインタ値そのものではなく Handle のポインタを渡すようにしている.
  
       * 最後の +6 は一時的な作業領域などとして使う分.)
      ---------------------------------------- -}

	  // Compute framesize for the wrapper.  We need to handlize all oops in
	  // incoming registers
	
	  // Calculate the total number of stack slots we will need.
	
	  // First count the abi requirement plus all of the outgoing args
	  int stack_slots = SharedRuntime::out_preserve_stack_slots() + out_arg_slots;
	
	  // Now the space for the inbound oop handle area
	
	  int oop_handle_offset = stack_slots;
	  stack_slots += 6*VMRegImpl::slots_per_word;
	
	  // Now any space we need for handlizing a klass if static method
	
	  int oop_temp_slot_offset = 0;
	  int klass_slot_offset = 0;
	  int klass_offset = -1;
	  int lock_slot_offset = 0;
	  bool is_static = false;
	
	  if (method->is_static()) {
	    klass_slot_offset = stack_slots;
	    stack_slots += VMRegImpl::slots_per_word;
	    klass_offset = klass_slot_offset * VMRegImpl::stack_slot_size;
	    is_static = true;
	  }
	
	  // Plus a lock if needed
	
	  if (method->is_synchronized()) {
	    lock_slot_offset = stack_slots;
	    stack_slots += VMRegImpl::slots_per_word;
	  }
	
	  // Now a place (+2) to save return values or temp during shuffling
	  // + 4 for return address (which we own) and saved rbp
	  stack_slots += 6;
	
	  // Ok The space we have allocated will look like:
	  //
	  //
	  // FP-> |                     |
	  //      |---------------------|
	  //      | 2 slots for moves   |
	  //      |---------------------|
	  //      | lock box (if sync)  |
	  //      |---------------------| <- lock_slot_offset
	  //      | klass (if static)   |
	  //      |---------------------| <- klass_slot_offset
	  //      | oopHandle area      |
	  //      |---------------------| <- oop_handle_offset (6 java arg registers)
	  //      | outbound memory     |
	  //      | based arguments     |
	  //      |                     |
	  //      |---------------------|
	  //      |                     |
	  // SP-> | out_preserved_slots |
	  //
	  //
	
	
	  // Now compute actual number of stack words we need rounding to make
	  // stack properly aligned.
	  stack_slots = round_to(stack_slots, StackAlignmentInSlots);
	
	  int stack_size = stack_slots * VMRegImpl::stack_slot_size;
	
	
  {- -------------------------------------------
  (1) コード生成:
      「Inline Caching 用の型検査を行う (= "unverified entry point" の処理). 
       型が違っていた場合は, SharedRuntime::get_ic_miss_stub() に格納されているスタブにジャンプする.
       合っていた場合は, このままフォールスルー」
      ---------------------------------------- -}

	  // First thing make an ic check to see if we should even be here
	
	  // We are free to use all registers as temps without saving them and
	  // restoring them except rbp. rbp is the only callee save register
	  // as far as the interpreter and the compiler(s) are concerned.
	
	
	  const Register ic_reg = rax;
	  const Register receiver = j_rarg0;
	
	  Label ok;
	  Label exception_pending;
	
	  assert_different_registers(ic_reg, receiver, rscratch1);
	  __ verify_oop(receiver);
	  __ load_klass(rscratch1, receiver);
	  __ cmpq(ic_reg, rscratch1);
	  __ jcc(Assembler::equal, ok);
	
	  __ jump(RuntimeAddress(SharedRuntime::get_ic_miss_stub()));
	
	  __ bind(ok);
	
	  // Verified entry point must be aligned
	  __ align(8);
	
  {- -------------------------------------------
  (1) (ここから先が "verified entry point"(vep))
      ---------------------------------------- -}

	  int vep_offset = ((intptr_t)__ pc()) - start;
	
  {- -------------------------------------------
  (1) コード生成: 
      (コメントによると, 
      vep のコードの先頭箇所には, 後で make_non_entrant にできるようにするため, 5バイトの空間が必要になる.
      stack banging コードを置いておくのにちょうどいいのでここにそのコードを生成する, 
      とのこと)
  
      * UseStackBanging オプションが指定されている場合:
        「MacroAssembler::bang_stack_with_offset() が生成するコードにより, 
         現在の SP から pages ページ分だけ離れたアドレスに書き込みを行う.」
    
      * UseStackBanging オプションが指定されていない場合:
        「なにもしない」 (5 バイトの nop を埋めておく)
      ---------------------------------------- -}

	  // The instruction at the verified entry point must be 5 bytes or longer
	  // because it can be patched on the fly by make_non_entrant. The stack bang
	  // instruction fits that requirement.
	
	  // Generate stack overflow check
	
	  if (UseStackBanging) {
	    __ bang_stack_with_offset(StackShadowPages*os::vm_page_size());
	  } else {
	    // need a 5 byte instruction to allow MT safe patching to non-entrant
	    __ fat_nop();
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「rbp をスタック上に待避し, rbp の値を rsp の値に変更」
      ---------------------------------------- -}

	  // Generate a new frame for the wrapper.
	  __ enter();

  {- -------------------------------------------
  (1) コード生成:
      「計算したスタックフレーム分だけ rsp をずらしておく
        (なお, -2 しているのはリターンアドレスは既にスタック上にあって不要なため)」
      ---------------------------------------- -}

	  // -2 because return address is already present and so is saved rbp
	  __ subptr(rsp, stack_size - 2*wordSize);
	
  {- -------------------------------------------
  (1) (ここから先が Frame_Complete)
      ---------------------------------------- -}

	    // Frame is now completed as far as size and linkage.
	
	    int frame_complete = ((intptr_t)__ pc()) - start;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	    {
	      Label L;
	      __ mov(rax, rsp);
	      __ andptr(rax, -16); // must be 16 byte boundary (see amd64 ABI)
	      __ cmpptr(rax, rsp);
	      __ jcc(Assembler::equal, L);
	      __ stop("improperly aligned stack");
	      __ bind(L);
	    }
	#endif /* ASSERT */
	
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // We use r14 as the oop handle for the receiver/klass
	  // It is callee save so it survives the call to native
	
	  const Register oop_handle_reg = r14;
	
	
	
  {- -------------------------------------------
  (1) コード生成:
      「引数を, ネイティブメソッドの calling convention に合わせて移動させる」
  
       (なお JNI 呼び出しでは, 普通の Java メソッドの呼び出し時と異なり, 
        引数として JNIEnv* を増やさないといけない (さらに static な場合には class 引数も付けないといけない).
        このため, 両方がレジスタ渡しの場合であっても位置は合わない)
  
        <= とはいえ個数は増えるだけなので, 後ろの方の引数から移動させていけば
       うっかり上書きするようなミスはありえない.
  
       (なお, OopMap も作って, どこにポインタを埋めたかも記録している.
        OopMap は(ちょっとした Trick だが)計算したスタックサイズの2倍(stack_slots * 2)の分を確保し,
        呼び出し元の incoming oop args に関する情報も格納できるようにしている模様)
        <= 最終的にできた OopMap は, nmethod のコンストラクタに渡して中に格納している模様.
      ---------------------------------------- -}

	  //
	  // We immediately shuffle the arguments so that any vm call we have to
	  // make from here on out (sync slow path, jvmti, etc.) we will have
	  // captured the oops from our caller and have a valid oopMap for
	  // them.
	
	  // -----------------
	  // The Grand Shuffle
	
	  // The Java calling convention is either equal (linux) or denser (win64) than the
	  // c calling convention. However the because of the jni_env argument the c calling
	  // convention always has at least one more (and two for static) arguments than Java.
	  // Therefore if we move the args from java -> c backwards then we will never have
	  // a register->register conflict and we don't have to build a dependency graph
	  // and figure out how to break any cycles.
	  //
	
	  // Record esp-based slot for receiver on stack for non-static methods
	  int receiver_offset = -1;
	
	  // This is a trick. We double the stack slots so we can claim
	  // the oops in the caller's frame. Since we are sure to have
	  // more args than the caller doubling is enough to make
	  // sure we can capture all the incoming oop args from the
	  // caller.
	  //
	  OopMap* map = new OopMap(stack_slots * 2, 0 /* arg_slots*/);
	
	  // Mark location of rbp (someday)
	  // map->set_callee_saved(VMRegImpl::stack2reg( stack_slots - 2), stack_slots * 2, 0, vmreg(rbp));
	
	  // Use eax, ebx as temporaries during any memory-memory moves we have to do
	  // All inbound args are referenced based on rbp and all outbound args via rsp.
	
	
	#ifdef ASSERT
	  bool reg_destroyed[RegisterImpl::number_of_registers];
	  bool freg_destroyed[XMMRegisterImpl::number_of_registers];
	  for ( int r = 0 ; r < RegisterImpl::number_of_registers ; r++ ) {
	    reg_destroyed[r] = false;
	  }
	  for ( int f = 0 ; f < XMMRegisterImpl::number_of_registers ; f++ ) {
	    freg_destroyed[f] = false;
	  }
	
	#endif /* ASSERT */
	
	
	  int c_arg = total_c_args - 1;
	  for ( int i = total_in_args - 1; i >= 0 ; i--, c_arg-- ) {
	#ifdef ASSERT
	    if (in_regs[i].first()->is_Register()) {
	      assert(!reg_destroyed[in_regs[i].first()->as_Register()->encoding()], "destroyed reg!");
	    } else if (in_regs[i].first()->is_XMMRegister()) {
	      assert(!freg_destroyed[in_regs[i].first()->as_XMMRegister()->encoding()], "destroyed reg!");
	    }
	    if (out_regs[c_arg].first()->is_Register()) {
	      reg_destroyed[out_regs[c_arg].first()->as_Register()->encoding()] = true;
	    } else if (out_regs[c_arg].first()->is_XMMRegister()) {
	      freg_destroyed[out_regs[c_arg].first()->as_XMMRegister()->encoding()] = true;
	    }
	#endif /* ASSERT */
	    switch (in_sig_bt[i]) {
	      case T_ARRAY:
	      case T_OBJECT:
	        object_move(masm, map, oop_handle_offset, stack_slots, in_regs[i], out_regs[c_arg],
	                    ((i == 0) && (!is_static)),
	                    &receiver_offset);
	        break;
	      case T_VOID:
	        break;
	
	      case T_FLOAT:
	        float_move(masm, in_regs[i], out_regs[c_arg]);
	          break;
	
	      case T_DOUBLE:
	        assert( i + 1 < total_in_args &&
	                in_sig_bt[i + 1] == T_VOID &&
	                out_sig_bt[c_arg+1] == T_VOID, "bad arg list");
	        double_move(masm, in_regs[i], out_regs[c_arg]);
	        break;
	
	      case T_LONG :
	        long_move(masm, in_regs[i], out_regs[c_arg]);
	        break;
	
	      case T_ADDRESS: assert(false, "found T_ADDRESS in java args");
	
	      default:
	        move32_64(masm, in_regs[i], out_regs[c_arg]);
	    }
	  }
	
	  // point c_arg at the first arg that is already loaded in case we
	  // need to spill before we call out
	  c_arg++;
	
  {- -------------------------------------------
  (1) コード生成: (対象のメソッドが staic method の場合にのみ生成)
      「引数として渡す必要があるので, クラスオブジェクトを O1 レジスタにセット
        (正確にはクラスオブジェクトを指す Handle を作り, O1 にその Handle をセット)」
      ---------------------------------------- -}

	  // Pre-load a static method's oop into r14.  Used both by locking code and
	  // the normal JNI call code.
	  if (method->is_static()) {
	
	    //  load oop into a register
	    __ movoop(oop_handle_reg, JNIHandles::make_local(Klass::cast(method->method_holder())->java_mirror()));
	
	    // Now handlize the static class mirror it's known not-null.
	    __ movptr(Address(rsp, klass_offset), oop_handle_reg);
	    map->set_oop(VMRegImpl::stack2reg(klass_slot_offset));
	
	    // Now get the handle
	    __ lea(oop_handle_reg, Address(rsp, klass_offset));
	    // store the klass handle as second argument
	    __ movptr(c_rarg1, oop_handle_reg);
	    // and protect the arg if we must spill
	    c_arg--;
	  }
	
  {- -------------------------------------------
  (1) この時点の pc を取得し, 作成した oopmap と対応付けて oopmapset に登録しておく.
      ---------------------------------------- -}

	  // Change state to native (we save the return address in the thread, since it might not
	  // be pushed on the stack when we do a a stack traversal). It is enough that the pc()
	  // points into the right code segment. It does not have to be the correct return pc.
	  // We use the same pc/oopMap repeatedly when we call out
	
	  intptr_t the_pc = (intptr_t) __ pc();
	  oop_maps->add_gc_map(the_pc - start, map);
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) を設定する.」 #TODO
      ---------------------------------------- -}

	  __ set_last_Java_frame(rsp, noreg, (address)the_pc);
	
	
  {- -------------------------------------------
  (1) コード生成: (DTrace のフック点) (See: SharedRuntime::dtrace_method_entry())
      ---------------------------------------- -}

	  // We have all of the arguments setup at this point. We must not touch any register
	  // argument registers at this point (what if we save/restore them there are no oop?
	
	  {
	    SkipIfEqual skip(masm, &DTraceMethodProbes, false);
	    // protect the args we've loaded
	    save_args(masm, total_c_args, c_arg, out_regs);
	    __ movoop(c_rarg1, JNIHandles::make_local(method()));
	    __ call_VM_leaf(
	      CAST_FROM_FN_PTR(address, SharedRuntime::dtrace_method_entry),
	      r15_thread, c_rarg1);
	    restore_args(masm, total_c_args, c_arg, out_regs);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (トレース出力) (See: SharedRuntime::rc_trace_method_entry())
      (JVMTI の RedefineClasses() 関係のトレース出力)
      ---------------------------------------- -}

	  // RedefineClasses() tracing support for obsolete method entry
	  if (RC_TRACE_IN_RANGE(0x00001000, 0x00002000)) {
	    // protect the args we've loaded
	    save_args(masm, total_c_args, c_arg, out_regs);
	    __ movoop(c_rarg1, JNIHandles::make_local(method()));
	    __ call_VM_leaf(
	      CAST_FROM_FN_PTR(address, SharedRuntime::rc_trace_method_entry),
	      r15_thread, c_rarg1);
	    restore_args(masm, total_c_args, c_arg, out_regs);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (対象のメソッドが synchronized method の場合にのみ生成)
      「ロックを取得する.
        (まず, ここで fast-path でのロック取得を試みる.
        失敗したら, slow_path_lock ラベルにジャンプして 
        SharedRuntime::complete_monitor_locking_C() でロックを取得した後, 
        またここに戻ってきて以降の処理を続ける)」
      ---------------------------------------- -}

	  // Lock a synchronized method
	
	  // Register definitions used by locking and unlocking
	
	  const Register swap_reg = rax;  // Must use rax for cmpxchg instruction
	  const Register obj_reg  = rbx;  // Will contain the oop
	  const Register lock_reg = r13;  // Address of compiler lock object (BasicLock)
	  const Register old_hdr  = r13;  // value of old header at unlock time
	
	  Label slow_path_lock;
	  Label lock_done;
	
	  if (method->is_synchronized()) {
	
	
	    const int mark_word_offset = BasicLock::displaced_header_offset_in_bytes();
	
	    // Get the handle (the 2nd argument)
	    __ mov(oop_handle_reg, c_rarg1);
	
	    // Get address of the box
	
	    __ lea(lock_reg, Address(rsp, lock_slot_offset * VMRegImpl::stack_slot_size));
	
	    // Load the oop from the handle
	    __ movptr(obj_reg, Address(oop_handle_reg, 0));
	
	    if (UseBiasedLocking) {
	      __ biased_locking_enter(lock_reg, obj_reg, swap_reg, rscratch1, false, lock_done, &slow_path_lock);
	    }
	
	    // Load immediate 1 into swap_reg %rax
	    __ movl(swap_reg, 1);
	
	    // Load (object->mark() | 1) into swap_reg %rax
	    __ orptr(swap_reg, Address(obj_reg, 0));
	
	    // Save (object->mark() | 1) into BasicLock's displaced header
	    __ movptr(Address(lock_reg, mark_word_offset), swap_reg);
	
	    if (os::is_MP()) {
	      __ lock();
	    }
	
	    // src -> dest iff dest == rax else rax <- dest
	    __ cmpxchgptr(lock_reg, Address(obj_reg, 0));
	    __ jcc(Assembler::equal, lock_done);
	
	    // Hmm should this move to the slow path code area???
	
	    // Test if the oopMark is an obvious stack pointer, i.e.,
	    //  1) (mark & 3) == 0, and
	    //  2) rsp <= mark < mark + os::pagesize()
	    // These 3 tests can be done by evaluating the following
	    // expression: ((mark - rsp) & (3 - os::vm_page_size())),
	    // assuming both stack pointer and pagesize have their
	    // least significant 2 bits clear.
	    // NOTE: the oopMark is in swap_reg %rax as the result of cmpxchg
	
	    __ subptr(swap_reg, rsp);
	    __ andptr(swap_reg, 3 - os::vm_page_size());
	
	    // Save the test result, for recursive case, the result is zero
	    __ movptr(Address(lock_reg, mark_word_offset), swap_reg);
	    __ jcc(Assembler::notEqual, slow_path_lock);
	
	    // Slow path will re-enter here
	
	    __ bind(lock_done);
	  }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「実際にネイティブメソッドを呼び出す処理を行う.
      
        なお, 呼び出す前に以下の処理を行っている.
        * JNIEnv* 引数をセットする
        * Thread の state を _thread_in_native に変更する  」
      ---------------------------------------- -}

	  // Finally just about ready to make the JNI call
	
	
	  // get JNIEnv* which is first argument to native
	
	  __ lea(c_rarg0, Address(r15_thread, in_bytes(JavaThread::jni_environment_offset())));
	
	  // Now set thread in native
	  __ movl(Address(r15_thread, JavaThread::thread_state_offset()), _thread_in_native);
	
	  __ call(RuntimeAddress(method->native_function()));
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Either restore the MXCSR register after returning from the JNI Call
	    // or verify that it wasn't changed.
	    if (RestoreMXCSROnJNICalls) {
	      __ ldmxcsr(ExternalAddress(StubRoutines::x86::mxcsr_std()));
	
	    }
	    else if (CheckJNICalls ) {
	      __ call(RuntimeAddress(CAST_FROM_FN_PTR(address, StubRoutines::x86::verify_mxcsr_entry())));
	    }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「ネイティブメソッドの返値を, Java VM 内の calling convention に合わせて移動させる」
      ---------------------------------------- -}

	  // Unpack native results.
	  switch (ret_type) {
	  case T_BOOLEAN: __ c2bool(rax);            break;
	  case T_CHAR   : __ movzwl(rax, rax);      break;
	  case T_BYTE   : __ sign_extend_byte (rax); break;
	  case T_SHORT  : __ sign_extend_short(rax); break;
	  case T_INT    : /* nothing to do */        break;
	  case T_DOUBLE :
	  case T_FLOAT  :
	    // Result is in xmm0 we'll save as needed
	    break;
	  case T_ARRAY:                 // Really a handle
	  case T_OBJECT:                // Really a handle
	      break; // can't de-handlize until after safepoint check
	  case T_VOID: break;
	  case T_LONG: break;
	  default       : ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの thread state (JavaThread::_thread_state フィールド) を, 
       _thread_in_native_trans に設定する.」
  
      なお, MP 環境の場合には
      VM Thread 側への visibility (sequential consistency) を保証する必要があるので, 
      変更後に以下のどちらかを行っている (See: SafepointSynchronize::begin()).
      * UseMembar オプションが指定されている場合:
        「OrderAccess::fence() が生成するコードでメモリバリアを張る.」
      * そうではない場合: 
        「InterfaceSupport::serialize_memory() が生成するコードで serialize page への書き込みを行う.」
  
      こうしないと, 以降の safepoint チェックが 
      VM Thread による _state の変更と race した場合に, すり抜けてしまう恐れがある.
      逆にこの順序で行うことで, race した場合でも VM Thread が見つけて
      この Thread の suspend_flags を変更してくれるようにできている.
      このため, SafepointSynchronize::_state と Thread::_suspend_flags の両方をチェックすれば, 
      すり抜けること無くチェックが行える.
      ---------------------------------------- -}

	  // Switch thread to "native transition" state before reading the synchronization state.
	  // This additional state is necessary because reading and testing the synchronization
	  // state is not atomic w.r.t. GC, as this scenario demonstrates:
	  //     Java thread A, in _thread_in_native state, loads _not_synchronized and is preempted.
	  //     VM thread changes sync state to synchronizing and suspends threads for GC.
	  //     Thread A is resumed to finish this native method, but doesn't block here since it
	  //     didn't see any synchronization is progress, and escapes.
	  __ movl(Address(r15_thread, JavaThread::thread_state_offset()), _thread_in_native_trans);
	
	  if(os::is_MP()) {
	    if (UseMembar) {
	      // Force this write out before the read below
	      __ membar(Assembler::Membar_mask_bits(
	           Assembler::LoadLoad | Assembler::LoadStore |
	           Assembler::StoreLoad | Assembler::StoreStore));
	    } else {
	      // Write serialization page so VM thread can do a pseudo remote membar.
	      // We use the current thread pointer to calculate a thread specific
	      // offset to write to within the page. This minimizes bus traffic
	      // due to cache line collision.
	      __ serialize_memory(r15_thread, rcx);
	    }
	  }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「もし Safepoint 停止が開始されていた場合, 
       JavaThread::check_special_condition_for_native_trans() を呼び出してここで block する. 」
      
       (確認は SafepointSynchronize::_state をチェックすることで行っている.)
      ---------------------------------------- -}

	  // check for safepoint operation in progress and/or pending suspend requests
	  {
	    Label Continue;
	
	    __ cmp32(ExternalAddress((address)SafepointSynchronize::address_of_state()),
	             SafepointSynchronize::_not_synchronized);
	
	    Label L;
	    __ jcc(Assembler::notEqual, L);
	    __ cmpl(Address(r15_thread, JavaThread::suspend_flags_offset()), 0);
	    __ jcc(Assembler::equal, Continue);
	    __ bind(L);
	
	    // Don't use call_VM as it will see a possible pending exception and forward it
	    // and never return here preventing us from clearing _last_native_pc down below.
	    // Also can't use call_VM_leaf either as it will check to see if rsi & rdi are
	    // preserved and correspond to the bcp/locals pointers. So we do a runtime call
	    // by hand.
	    //
	    save_native_result(masm, ret_type, stack_slots);
	    __ mov(c_rarg0, r15_thread);
	    __ mov(r12, rsp); // remember sp
	    __ subptr(rsp, frame::arg_reg_save_area_bytes); // windows
	    __ andptr(rsp, -16); // align stack as required by ABI
	    __ call(RuntimeAddress(CAST_FROM_FN_PTR(address, JavaThread::check_special_condition_for_native_trans)));
	    __ mov(rsp, r12); // restore sp
	    __ reinit_heapbase();
	    // Restore any method result value
	    restore_native_result(masm, ret_type, stack_slots);
	    __ bind(Continue);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「(この時点では thread_in_native_trans になっているはずなので)
        カレントスレッドの thread state (JavaThread::_thread_state フィールド) を _thread_in_Java に戻しておく.」
      ---------------------------------------- -}

	  // change thread state
	  __ movl(Address(r15_thread, JavaThread::thread_state_offset()), _thread_in_Java);
	
  {- -------------------------------------------
  (1) コード生成:
      「スタック上の yellow page が disabled になっていたら 
       reguard ラベルにジャンプして SharedRuntime::reguard_yellow_pages() で元に戻した後, 
       またここに戻ってきて以降の処理を続ける」
      ---------------------------------------- -}

	  Label reguard;
	  Label reguard_done;
	  __ cmpl(Address(r15_thread, JavaThread::stack_guard_state_offset()), JavaThread::stack_guard_yellow_disabled);
	  __ jcc(Assembler::equal, reguard);
	  __ bind(reguard_done);
	
  {- -------------------------------------------
  (1) コード生成: (対象のメソッドが synchronized method の場合にのみ生成)
      「取得したロックを解放しておく.
        (まず, ここで fast-path でのロック解放を試みる.
        失敗したら, slow_path_unlock ラベルにジャンプして 
        SharedRuntime::complete_monitor_unlocking_C() でロックを取得した後, 
        またここに戻ってきて以降の処理を続ける)」
      ---------------------------------------- -}

	  // native result if any is live
	
	  // Unlock
	  Label unlock_done;
	  Label slow_path_unlock;
	  if (method->is_synchronized()) {
	
	    // Get locked oop from the handle we passed to jni
	    __ movptr(obj_reg, Address(oop_handle_reg, 0));
	
	    Label done;
	
	    if (UseBiasedLocking) {
	      __ biased_locking_exit(obj_reg, old_hdr, done);
	    }
	
	    // Simple recursive lock?
	
	    __ cmpptr(Address(rsp, lock_slot_offset * VMRegImpl::stack_slot_size), (int32_t)NULL_WORD);
	    __ jcc(Assembler::equal, done);
	
	    // Must save rax if if it is live now because cmpxchg must use it
	    if (ret_type != T_FLOAT && ret_type != T_DOUBLE && ret_type != T_VOID) {
	      save_native_result(masm, ret_type, stack_slots);
	    }
	
	
	    // get address of the stack lock
	    __ lea(rax, Address(rsp, lock_slot_offset * VMRegImpl::stack_slot_size));
	    //  get old displaced header
	    __ movptr(old_hdr, Address(rax, 0));
	
	    // Atomic swap old header if oop still contains the stack lock
	    if (os::is_MP()) {
	      __ lock();
	    }
	    __ cmpxchgptr(old_hdr, Address(obj_reg, 0));
	    __ jcc(Assembler::notEqual, slow_path_unlock);
	
	    // slow path re-enters here
	    __ bind(unlock_done);
	    if (ret_type != T_FLOAT && ret_type != T_DOUBLE && ret_type != T_VOID) {
	      restore_native_result(masm, ret_type, stack_slots);
	    }
	
	    __ bind(done);
	
	  }

  {- -------------------------------------------
  (1) コード生成: (DTrace のフック点) (See: SharedRuntime::dtrace_method_exit())
      ---------------------------------------- -}

	  {
	    SkipIfEqual skip(masm, &DTraceMethodProbes, false);
	    save_native_result(masm, ret_type, stack_slots);
	    __ movoop(c_rarg1, JNIHandles::make_local(method()));
	    __ call_VM_leaf(
	         CAST_FROM_FN_PTR(address, SharedRuntime::dtrace_method_exit),
	         r15_thread, c_rarg1);
	    restore_native_result(masm, ret_type, stack_slots);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) をクリアする」
      ---------------------------------------- -}

	  __ reset_last_Java_frame(false, true);
	
  {- -------------------------------------------
  (1) コード生成: (返値の型が oop(オブジェクトや配列) の場合にのみ生成)
      「返値は JNIHandle で返されるので, 中身を取り出しておく.
       (ただし返値が NULL の場合は NULL のままにしておく)」
      ---------------------------------------- -}

	  // Unpack oop result
	  if (ret_type == T_OBJECT || ret_type == T_ARRAY) {
	      Label L;
	      __ testptr(rax, rax);
	      __ jcc(Assembler::zero, L);
	      __ movptr(rax, Address(rax, 0));
	      __ bind(L);
	      __ verify_oop(rax);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「JNI local handles を解放する.
       (もっと具体的に言うと, JNIHandleBlock::_top の値を 0 に変更する. 
        これにより JNIHandleBlock からの参照が無くなるため, GC で回収されるようになる.)」
      ---------------------------------------- -}

	  // reset handle block
	  __ movptr(rcx, Address(r15_thread, JavaThread::active_handles_offset()));
	  __ movptr(Address(rcx, JNIHandleBlock::top_offset_in_bytes()), (int32_t)NULL_WORD);
	
  {- -------------------------------------------
  (1) コード生成:
      「スタックフレームを破棄する」
      ---------------------------------------- -}

	  // pop our frame
	
	  __ leave();
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの pending_exception フィールドを確認し
        ネイティブメソッド呼び出し中に例外が発生したかどうかをチェックする. 
        もし 0 でなければ, 例外が発生したということなので 
        exception_pending ラベルに飛んで 
        StubRoutines::forward_exception_entry() が指しているコードを呼び出す.」
      ---------------------------------------- -}

	  // Any exception pending?
	  __ cmpptr(Address(r15_thread, in_bytes(Thread::pending_exception_offset())), (int32_t)NULL_WORD);
	  __ jcc(Assembler::notEqual, exception_pending);
	
  {- -------------------------------------------
  (1) コード生成:
      「リターンする」
      ---------------------------------------- -}

	  // Return
	
	  __ ret(0);
	
  {- -------------------------------------------
  (1) (これ以降は, 実行頻度が低いと思われるコードを集めた領域)
      ---------------------------------------- -}

	  // Unexpected paths are out of line and go here
	
  {- -------------------------------------------
  (1) コード生成:
    「(ここが exception_pending ラベルの位置)」
    「StubRoutines::forward_exception_entry() が指しているコードを呼び出す」
      ---------------------------------------- -}

	  // forward the exception
	  __ bind(exception_pending);
	
	  // and forward the exception
	  __ jump(RuntimeAddress(StubRoutines::forward_exception_entry()));
	
	
  {- -------------------------------------------
  (1) コード生成: (対象のメソッドが synchronized method の場合にのみ生成)
      「(ここが slow_path_lock ラベル, 及び slow_path_unlock ラベル の位置)」
      (これらは, 上記の fast-path でのロック取得／ロック解放が失敗した場合のパス)
  
      * slow_path_lock ラベルでの処理
       「SharedRuntime::complete_monitor_locking_C() を呼び出してロックを取得した後, 
         lock_done ラベルに戻って処理を続ける」
  
      * slow_path_unlock ラベルでの処理
       「SharedRuntime::complete_monitor_unlocking_C() を呼び出してロックを解放した後, 
         unlock_done ラベルに戻って処理を続ける」
      ---------------------------------------- -}

	  // Slow path locking & unlocking
	  if (method->is_synchronized()) {
	
	    // BEGIN Slow path lock
	    __ bind(slow_path_lock);
	
	    // has last_Java_frame setup. No exceptions so do vanilla call not call_VM
	    // args are (oop obj, BasicLock* lock, JavaThread* thread)
	
	    // protect the args we've loaded
	    save_args(masm, total_c_args, c_arg, out_regs);
	
	    __ mov(c_rarg0, obj_reg);
	    __ mov(c_rarg1, lock_reg);
	    __ mov(c_rarg2, r15_thread);
	
	    // Not a leaf but we have last_Java_frame setup as we want
	    __ call_VM_leaf(CAST_FROM_FN_PTR(address, SharedRuntime::complete_monitor_locking_C), 3);
	    restore_args(masm, total_c_args, c_arg, out_regs);
	
	#ifdef ASSERT
	    { Label L;
	    __ cmpptr(Address(r15_thread, in_bytes(Thread::pending_exception_offset())), (int32_t)NULL_WORD);
	    __ jcc(Assembler::equal, L);
	    __ stop("no pending exception allowed on exit from monitorenter");
	    __ bind(L);
	    }
	#endif
	    __ jmp(lock_done);
	
	    // END Slow path lock
	
	    // BEGIN Slow path unlock
	    __ bind(slow_path_unlock);
	
	    // If we haven't already saved the native result we must save it now as xmm registers
	    // are still exposed.
	
	    if (ret_type == T_FLOAT || ret_type == T_DOUBLE ) {
	      save_native_result(masm, ret_type, stack_slots);
	    }
	
	    __ lea(c_rarg1, Address(rsp, lock_slot_offset * VMRegImpl::stack_slot_size));
	
	    __ mov(c_rarg0, obj_reg);
	    __ mov(r12, rsp); // remember sp
	    __ subptr(rsp, frame::arg_reg_save_area_bytes); // windows
	    __ andptr(rsp, -16); // align stack as required by ABI
	
	    // Save pending exception around call to VM (which contains an EXCEPTION_MARK)
	    // NOTE that obj_reg == rbx currently
	    __ movptr(rbx, Address(r15_thread, in_bytes(Thread::pending_exception_offset())));
	    __ movptr(Address(r15_thread, in_bytes(Thread::pending_exception_offset())), (int32_t)NULL_WORD);
	
	    __ call(RuntimeAddress(CAST_FROM_FN_PTR(address, SharedRuntime::complete_monitor_unlocking_C)));
	    __ mov(rsp, r12); // restore sp
	    __ reinit_heapbase();
	#ifdef ASSERT
	    {
	      Label L;
	      __ cmpptr(Address(r15_thread, in_bytes(Thread::pending_exception_offset())), (int)NULL_WORD);
	      __ jcc(Assembler::equal, L);
	      __ stop("no pending exception allowed on exit complete_monitor_unlocking_C");
	      __ bind(L);
	    }
	#endif /* ASSERT */
	
	    __ movptr(Address(r15_thread, in_bytes(Thread::pending_exception_offset())), rbx);
	
	    if (ret_type == T_FLOAT || ret_type == T_DOUBLE ) {
	      restore_native_result(masm, ret_type, stack_slots);
	    }
	    __ jmp(unlock_done);
	
	    // END Slow path unlock
	
	  } // synchronized
	
  {- -------------------------------------------
  (1) コード生成:
      「(ここが reguard ラベルの位置)」
      「SharedRuntime::reguard_yellow_pages() を呼んで
        スタック上の yellow page を元に戻し, 
        reguard_done ラベルに戻って処理を続ける.」
      ---------------------------------------- -}

	  // SLOW PATH Reguard the stack if needed
	
	  __ bind(reguard);
	  save_native_result(masm, ret_type, stack_slots);
	  __ mov(r12, rsp); // remember sp
	  __ subptr(rsp, frame::arg_reg_save_area_bytes); // windows
	  __ andptr(rsp, -16); // align stack as required by ABI
	  __ call(RuntimeAddress(CAST_FROM_FN_PTR(address, SharedRuntime::reguard_yellow_pages)));
	  __ mov(rsp, r12); // restore sp
	  __ reinit_heapbase();
	  restore_native_result(masm, ret_type, stack_slots);
	  // and continue
	  __ jmp(reguard_done);
	
	
	
  {- -------------------------------------------
  (1) 以上のコードを flush.
      ---------------------------------------- -}

	  __ flush();
	
  {- -------------------------------------------
  (1) nmethod::new_native_nmethod() を呼んで, できたコードを登録する
      ---------------------------------------- -}

	  nmethod *nm = nmethod::new_native_nmethod(method,
	                                            compile_id,
	                                            masm->code(),
	                                            vep_offset,
	                                            frame_complete,
	                                            stack_slots / VMRegImpl::slots_per_word,
	                                            (is_static ? in_ByteSize(klass_offset) : in_ByteSize(receiver_offset)),
	                                            in_ByteSize(lock_slot_offset*VMRegImpl::stack_slot_size),
	                                            oop_maps);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return nm;
	
	}
	
```


