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
//
// Interpreter stub for calling a native method. (asm interpreter)
// This sets up a somewhat different looking stack for calling the native method
// than the typical interpreter frame setup.
//

```

### 名前(function name)
```
address InterpreterGenerator::generate_native_entry(bool synchronized) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (G3 と G1 レジスタを作業用のレジスタとして使用)
      ---------------------------------------- -}

	  // the following temporary registers are used during frame creation
	  const Register Gtmp1 = G3_scratch ;
	  const Register Gtmp2 = G1_scratch;
	  bool inc_counter  = UseCompiler || CountCompiledCalls;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // make sure registers are different!
	  assert_different_registers(G2_thread, G5_method, Gargs, Gtmp1, Gtmp2);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Address Laccess_flags(Lmethod, methodOopDesc::access_flags_offset());
	
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

	  // make sure method is native & not abstract
	  // rethink these assertions - they can be simplified and shared (gri 2/25/2000)
	#ifdef ASSERT
	  __ ld(G5_method, methodOopDesc::access_flags_offset(), Gtmp1);
	  {
	    Label L;
	    __ btst(JVM_ACC_NATIVE, Gtmp1);
	    __ br(Assembler::notZero, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ stop("tried to execute non-native method as native");
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
	  generate_fixed_frame(true);
	
  {- -------------------------------------------
  (1) (Java メソッド用のエントリの場合には, 局所変数用の領域を 
       0 クリアするコードがここに入っているが, 
       この関数はネイティブメソッド用なので必要ない.
  
       See: InterpreterGenerator::generate_normal_entry())
      ---------------------------------------- -}

	  //
	  // No locals to initialize for native method
	  //
	
  {- -------------------------------------------
  (1) コード生成:
      「スタックフレーム中の interpreter_frame_oop_temp_offset の箇所を 0 クリアしておく」
      
      (interpreter_frame_oop_temp_offset は 
       ネイティブメソッドがポインタを返す場合にその返値を書き込む場所.
       変な値が入っていると, 途中で GC が実行された場合に
       それをポインタ値として解釈されて誤動作する恐れがあるため, NULL でクリアしておく)
      ---------------------------------------- -}

	  // this slot will be set later, we initialize it to null here just in
	  // case we get a GC before the actual value is stored later
	  __ st_ptr(G0, FP, (frame::interpreter_frame_oop_temp_offset * wordSize) + STACK_BIAS);
	
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
	  // This flag has two effects, one is to force an unwind in the topmost
	  // interpreter frame and not perform an unlock while doing so.
	
	  __ movbool(true, G3_scratch);
	  __ stbool(G3_scratch, do_not_unlock_if_synchronized);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, JIT コンパイルのためにメソッドの呼び出し回数を数える必要がある場合のみ生成する. 
                 より具体的には, UseCompiler オプションまたは CountCompiledCalls オプションが指定されている場合のみ生成)
  
      「InterpreterGenerator::generate_counter_incr() が生成するコードで, 
        invocation counter 値の増加処理と値のチェック処理を行う.
   
        値に応じて, 以下の2通りに分岐.
        * JIT コンパイル対象になる閾値を超えている場合:
          invocation_counter_overflow ラベルまでジャンプ
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
	  Label Lcontinue;
	  if (inc_counter) {
	    generate_counter_incr(&invocation_counter_overflow, NULL, NULL);
	
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

	  bang_stack_shadow_pages(true);
	
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
	      __ ld(Laccess_flags, O0);
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

	  // JVMTI support
	  __ notify_method_entry();
	
  {- -------------------------------------------
  (1) (以下で, 実際にネイティブコールの呼び出し処理を行う)
      ---------------------------------------- -}

	  // native call
	
	  // (note that O0 is never an oop--at most it is a handle)
	  // It is important not to smash any handles created by this call,
	  // until any oop handle in O0 is dereferenced.
	
	  // (note that the space for outgoing params is preallocated)
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し先の methodOop オブジェクト内から (signature_handler フィールドに格納されている) シグネチャハンドラを取得する.
        もし signature_handler フィールドが空であれば, InterpreterRuntime::prepare_native_call() を呼び出して
        methodOop オブジェクトにシグネチャハンドラを設定した後で, 改めて取得する.」
      ---------------------------------------- -}

	  // get signature handler
	  { Label L;
	    Address signature_handler(Lmethod, methodOopDesc::signature_handler_offset());
	    __ ld_ptr(signature_handler, G3_scratch);
	    __ tst(G3_scratch);
	    __ brx(Assembler::notZero, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::prepare_native_call), Lmethod);
	    __ ld_ptr(signature_handler, G3_scratch);
	    __ bind(L);
	  }
	
  {- -------------------------------------------
  (1) (#TODO)
       一つ frame をつんで register を退避 (Lmethod と Llocals だけは, コピーしておく)  
  
      (なお, mirror handle とは handle を格納する領域)
      ---------------------------------------- -}

	  // Push a new frame so that the args will really be stored in
	  // Copy a few locals across so the new frame has the variables
	  // we need but these values will be dead at the jni call and
	  // therefore not gc volatile like the values in the current
	  // frame (Lmethod in particular)
	
	  // Flush the method pointer to the register save area
	  __ st_ptr(Lmethod, SP, (Lmethod->sp_offset_in_saved_window() * wordSize) + STACK_BIAS);
	  __ mov(Llocals, O1);
	
	  // calculate where the mirror handle body is allocated in the interpreter frame:
	  __ add(FP, (frame::interpreter_frame_oop_temp_offset * wordSize) + STACK_BIAS, O2);
	
	  // Calculate current frame size
	  __ sub(SP, FP, O3);         // Calculate negative of current frame size
	  __ save(SP, O3, SP);        // Allocate an identical sized frame
	
	  // Note I7 has leftover trash. Slow signature handler will fill it in
	  // should we get there. Normal jni call will set reasonable last_Java_pc
	  // below (and fix I7 so the stack trace doesn't have a meaningless frame
	  // in it).
	
  {- -------------------------------------------
  (1) コード生成:
      「(save した後だが) Lmethod だけは save した位置からレジスタに復帰させておく.
        また, Llocals も, さっき待避した O1 から復帰させておく (save 後なので I1 だが).
        ついでに, Lscratch2 に mirror のアドレスを入れておく.」  #TODO
  
      (下記のコメントによると, この段階では Lmethod レジスタと Llocals レジスタのみが正しい値になっている, とのこと)
      ---------------------------------------- -}

	  // Load interpreter frame's Lmethod into same register here
	
	  __ ld_ptr(FP, (Lmethod->sp_offset_in_saved_window() * wordSize) + STACK_BIAS, Lmethod);
	
	  __ mov(I1, Llocals);
	  __ mov(I2, Lscratch2);     // save the address of the mirror
	
	
	  // ONLY Lmethod and Llocals are valid here!
	
  {- -------------------------------------------
  (1) コード生成:
      「シグネチャハンドラを呼び出す.
        (これによって, Llocals が指す位置(インタープリタのオペランドスタック内)に格納されている引数を
        ネイティブメソッドの ABI で規定されているレジスタやスタック上にコピーする)」
      ---------------------------------------- -}

	  // call signature handler, It will move the arg properly since Llocals in current frame
	  // matches that in outer frame
	
	  __ callr(G3_scratch, 0);
	  __ delayed()->nop();
	
	  // Result handler is in Lscratch
	
  {- -------------------------------------------
  (1) コード生成:
      「Lmethod をもう一度 save した位置からレジスタに復帰させておく.」 #TODO
      ---------------------------------------- -}

	  // Reload interpreter frame's Lmethod since slow signature handler may block
	  __ ld_ptr(FP, (Lmethod->sp_offset_in_saved_window() * wordSize) + STACK_BIAS, Lmethod);
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し先の methodOop 内の native_function_offset から, 
        ネイティブメソッドのエントリポイントのアドレスを取得する.      
  
        呼び出し先の methodOop 内を見て static メソッドかどうかを判定する.
        static メソッドであれば, クラスオブジェクト(mirror object)を取得して Handle でくるみ, 引数は Handle にする.
        (static メソッドでなければ何もしない)」
      ---------------------------------------- -}

	  { Label not_static;
	
	    __ ld(Laccess_flags, O0);
	    __ btst(JVM_ACC_STATIC, O0);
	    __ br( Assembler::zero, false, Assembler::pt, not_static);
	    // get native function entry point(O0 is a good temp until the very end)
	    __ delayed()->ld_ptr(Lmethod, in_bytes(methodOopDesc::native_function_offset()), O0);
	    // for static methods insert the mirror argument
	    const int mirror_offset = klassOopDesc::klass_part_offset_in_bytes() + Klass::java_mirror_offset_in_bytes();
	
	    __ ld_ptr(Lmethod, methodOopDesc:: constants_offset(), O1);
	    __ ld_ptr(O1, constantPoolOopDesc::pool_holder_offset_in_bytes(), O1);
	    __ ld_ptr(O1, mirror_offset, O1);
	#ifdef ASSERT
	    if (!PrintSignatureHandlers)  // do not dirty the output with this
	    { Label L;
	      __ tst(O1);
	      __ brx(Assembler::notZero, false, Assembler::pt, L);
	      __ delayed()->nop();
	      __ stop("mirror is missing");
	      __ bind(L);
	    }
	#endif // ASSERT
	    __ st_ptr(O1, Lscratch2, 0);
	    __ mov(Lscratch2, O1);
	    __ bind(not_static);
	  }
	
  {- -------------------------------------------
  (1) (この段階では, 呼び出し先のネイティブメソッドの引数も ABI に従った形で配置し終わっており, 
       また oop の引数は handle で包んである.
       result handler は Lscratch に入っている.)
      ---------------------------------------- -}

	  // At this point, arguments have been copied off of stack into
	  // their JNI positions, which are O1..O5 and SP[68..].
	  // Oops are boxed in-place on the stack, with handles copied to arguments.
	  // The result handler is in Lscratch.  O0 will shortly hold the JNIEnv*.
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  { Label L;
	    __ tst(O0);
	    __ brx(Assembler::notZero, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ stop("native entry point is missing");
	    __ bind(L);
	  }
	#endif // ASSERT
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) を設定する.」 #TODO
      ---------------------------------------- -}

	  //
	  // setup the frame anchor
	  //
	  // The scavenge function only needs to know that the PC of this frame is
	  // in the interpreter method entry code, it doesn't need to know the exact
	  // PC and hence we can use O7 which points to the return address from the
	  // previous call in the code stream (signature handler function)
	  //
	  // The other trick is we set last_Java_sp to FP instead of the usual SP because
	  // we have pushed the extra frame in order to protect the volatile register(s)
	  // in that frame when we return from the jni call
	  //
	
	  __ set_last_Java_frame(FP, O7);
	  __ mov(O7, I7);  // make dummy interpreter frame look like one above,
	                   // not meaningless information that'll confuse me.
	
  {- -------------------------------------------
  (1) コード生成:
      「」 ?? #TODO
      ---------------------------------------- -}

	  // flush the windows now. We don't care about the current (protection) frame
	  // only the outer frames
	
	  __ flush_windows();
	
	  // mark windows as flushed
	  Address flags(G2_thread, JavaThread::frame_anchor_offset() + JavaFrameAnchor::flags_offset());
	  __ set(JavaFrameAnchor::flushed, G3_scratch);
	  __ st(G3_scratch, flags);
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの thread state (JavaThread::_thread_state フィールド) を, 
       _thread_in_native に設定する.」
      ---------------------------------------- -}

	  // Transition from _thread_in_Java to _thread_in_native. We are already safepoint ready.
	
	  Address thread_state(G2_thread, JavaThread::thread_state_offset());
	#ifdef ASSERT
	  { Label L;
	    __ ld(thread_state, G3_scratch);
	    __ cmp(G3_scratch, _thread_in_Java);
	    __ br(Assembler::equal, false, Assembler::pt, L);
	    __ delayed()->nop();
	    __ stop("Wrong thread state in native stub");
	    __ bind(L);
	  }
	#endif // ASSERT
	  __ set(_thread_in_native, G3_scratch);
	  __ st(G3_scratch, thread_state);
	
  {- -------------------------------------------
  (1) コード生成:
      「O0 に最後の引数 (JNIEnv*) を設定しつつ, ネイティブメソッドのエントリポイントにジャンプする.」
       (なお, ジャンプの直前までは O0 に飛び先アドレスが入っている)
      ---------------------------------------- -}

	  // Call the jni method, using the delay slot to set the JNIEnv* argument.
	  __ save_thread(L7_thread_cache); // save Gthread
	  __ callr(O0, 0);
	  __ delayed()->
	     add(L7_thread_cache, in_bytes(JavaThread::jni_environment_offset()), O0);
	
  {- -------------------------------------------
  (1) (以下, ネイティブメソッドからリターンしてきた後の後始末処理)
      ---------------------------------------- -}

	  // Back from jni method Lmethod in this frame is DEAD, DEAD, DEAD
	
  {- -------------------------------------------
  (1) コード生成:
      「G2 レジスタ, 及び G6 レジスタの値を復元する.」
      ---------------------------------------- -}

	  __ restore_thread(L7_thread_cache); // restore G2_thread
	  __ reinit_heapbase();
	
  {- -------------------------------------------
  (1) コード生成:
      「もし Safepoint 停止が開始されていた場合, 
       JavaThread::check_special_condition_for_native_trans() を呼び出してここで block する. 」
      
       (確認は SafepointSynchronize::_state をチェックすることで行っている.)
      
       (なお, チェックの直前に Thread の state を _thread_in_native_trans に変えておく
        さらに, MP 環境の場合には
        VM Thread 側への visibility (sequential consistency) を保証する必要があるので, 
        変更後に以下のどちらかを行っている (See: SafepointSynchronize::begin()).
        * UseMembar オプションが指定されている場合:
          「OrderAccess::fence() が生成するコードでメモリバリアを張る.」
        * そうではない場合: 
          「InterfaceSupport::serialize_memory() が生成するコードで serialize page への書き込みを行う.」
  
        こうしないと, チェックが VM Thread による _state の変更と race した場合に, すり抜けてしまう恐れがある.
        逆にこの順序で行うことで, race した場合でも VM Thread が見つけて
        この Thread の suspend_flags を変更してくれるようにできている.
        このため, SafepointSynchronize::_state と Thread::_suspend_flags の両方をチェックすれば, 
        すり抜けること無くチェックが行える)
      ---------------------------------------- -}

	  // must we block?
	
	  // Block, if necessary, before resuming in _thread_in_Java state.
	  // In order for GC to work, don't clear the last_Java_sp until after blocking.
	  { Label no_block;
	    AddressLiteral sync_state(SafepointSynchronize::address_of_state());
	
	    // Switch thread to "native transition" state before reading the synchronization state.
	    // This additional state is necessary because reading and testing the synchronization
	    // state is not atomic w.r.t. GC, as this scenario demonstrates:
	    //     Java thread A, in _thread_in_native state, loads _not_synchronized and is preempted.
	    //     VM thread changes sync state to synchronizing and suspends threads for GC.
	    //     Thread A is resumed to finish this native method, but doesn't block here since it
	    //     didn't see any synchronization is progress, and escapes.
	    __ set(_thread_in_native_trans, G3_scratch);
	    __ st(G3_scratch, thread_state);
	    if(os::is_MP()) {
	      if (UseMembar) {
	        // Force this write out before the read below
	        __ membar(Assembler::StoreLoad);
	      } else {
	        // Write serialization page so VM thread can do a pseudo remote membar.
	        // We use the current thread pointer to calculate a thread specific
	        // offset to write to within the page. This minimizes bus traffic
	        // due to cache line collision.
	        __ serialize_memory(G2_thread, G1_scratch, G3_scratch);
	      }
	    }
	    __ load_contents(sync_state, G3_scratch);
	    __ cmp(G3_scratch, SafepointSynchronize::_not_synchronized);
	
	    Label L;
	    __ br(Assembler::notEqual, false, Assembler::pn, L);
	    __ delayed()->ld(G2_thread, JavaThread::suspend_flags_offset(), G3_scratch);
	    __ cmp(G3_scratch, 0);
	    __ br(Assembler::equal, false, Assembler::pt, no_block);
	    __ delayed()->nop();
	    __ bind(L);
	
	    // Block.  Save any potential method result value before the operation and
	    // use a leaf call to leave the last_Java_frame setup undisturbed.
	    save_native_result();
	    __ call_VM_leaf(L7_thread_cache,
	                    CAST_FROM_FN_PTR(address, JavaThread::check_special_condition_for_native_trans),
	                    G2_thread);
	
	    // Restore any method result value
	    restore_native_result();
	    __ bind(no_block);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの java frame anchor (Thread::_anchor フィールド) をクリアする」
      ---------------------------------------- -}

	  // Clear the frame anchor now
	
	  __ reset_last_Java_frame();
	
  {- -------------------------------------------
  (1) コード生成:
      「restore 命令を実行して, 待避していたレジスタを復帰させる.
  
       (ただし, result handler が入っている Lscratch と
        返り値が入っている O0(32bitではO1も) だけは, restore 後も同じ値にしておく)」
      ---------------------------------------- -}

	  // Move the result handler address
	  __ mov(Lscratch, G3_scratch);
	  // return possible result to the outer frame
	#ifndef __LP64
	  __ mov(O0, I0);
	  __ restore(O1, G0, O1);
	#else
	  __ restore(O0, G0, O0);
	#endif /* __LP64 */
	
	  // Move result handler to expected register
	  __ mov(G3_scratch, Lscratch);
	
  {- -------------------------------------------
  (1) コード生成:
      「(この時点では thread_in_native_trans になっているはずなので)
        カレントスレッドの thread state (JavaThread::_thread_state フィールド) を _thread_in_Java に戻しておく.」
      ---------------------------------------- -}

	  // Back in normal (native) interpreter frame. State is thread_in_native_trans
	  // switch to thread_in_Java.
	
	  __ set(_thread_in_Java, G3_scratch);
	  __ st(G3_scratch, thread_state);
	
  {- -------------------------------------------
  (1) コード生成:
      「JNI local handles を解放する.
       (もっと具体的に言うと, JNIHandleBlock::_top の値を 0 に変更する. 
        これにより JNIHandleBlock からの参照が無くなるため, GC で回収されるようになる.)」
      ---------------------------------------- -}

	  // reset handle block
	  __ ld_ptr(G2_thread, JavaThread::active_handles_offset(), G3_scratch);
	  __ st_ptr(G0, G3_scratch, JNIHandleBlock::top_offset_in_bytes());
	
  {- -------------------------------------------
  (1) コード生成:
      「もし返り値が oop 型で, かつ値も null でなければ, 
        (GC の調査対象に入れておかないと勝手に dangling pointer にされてしまうかもしれないので)
       スタックフレーム中の frame::interpreter_frame_oop_temp_offset の位置にコピーしておく.」
      ---------------------------------------- -}

	  // If we have an oop result store it where it will be safe for any further gc
	  // until we return now that we've released the handle it might be protected by
	
	  {
	    Label no_oop, store_result;
	
	    __ set((intptr_t)AbstractInterpreter::result_handler(T_OBJECT), G3_scratch);
	    __ cmp(G3_scratch, Lscratch);
	    __ brx(Assembler::notEqual, false, Assembler::pt, no_oop);
	    __ delayed()->nop();
	    __ addcc(G0, O0, O0);
	    __ brx(Assembler::notZero, true, Assembler::pt, store_result);     // if result is not NULL:
	    __ delayed()->ld_ptr(O0, 0, O0);                                   // unbox it
	    __ mov(G0, O0);
	
	    __ bind(store_result);
	    // Store it where gc will look for it and result handler expects it.
	    __ st_ptr(O0, FP, (frame::interpreter_frame_oop_temp_offset*wordSize) + STACK_BIAS);
	
	    __ bind(no_oop);
	
	  }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの pending_exception フィールドを確認し, 
        ネイティブメソッド呼び出し中に例外が発生したかどうかをチェックする. 
        もし 0 でなければ, 例外が発生したということなので 
        MacroAssembler::call_VM() が生成するコードでランタイムを呼び出して伝搬させる.
  
        (呼び出し先は InterpreterRuntime::throw_pending_exception() だが, 
         MacroAssembler::call_VM() のチェックコードで引っかけるのが目的.
         InterpreterRuntime::throw_pending_exception() 自体は何もしない.)」
       (See: [here](no3059d4M.html) for details)
      ---------------------------------------- -}

	  // handle exceptions (exception handling will handle unlocking!)
	  { Label L;
	    Address exception_addr(G2_thread, Thread::pending_exception_offset());
	    __ ld_ptr(exception_addr, Gtemp);
	    __ tst(Gtemp);
	    __ brx(Assembler::equal, false, Assembler::pt, L);
	    __ delayed()->nop();
	    // Note: This could be handled more efficiently since we know that the native
	    //       method doesn't have an exception handler. We could directly return
	    //       to the exception handler for the caller.
	    __ call_VM(noreg, CAST_FROM_FN_PTR(address, InterpreterRuntime::throw_pending_exception));
	    __ should_not_reach_here();
	    __ bind(L);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  // JVMTI support (preserves thread register)
	  __ notify_method_exit(true, ilgl, InterpreterMacroAssembler::NotifyJVMTI);
	
  {- -------------------------------------------
  (1) コード生成: (引数で synchronized method だと指定されている場合にのみ生成)
      「InterpreterGenerator::unlock_method() が生成するコードで, ロックを開放する」
      (See: [here](no3059F5A.html) for details)
    
      (なお, この処理の前後で, 以下の関数が生成するコードにより
       ネイティブメソッドの返値の待避復帰を行っている.
       * InterpreterGenerator::save_native_result()
       * InterpreterGenerator::restore_native_result())
      ---------------------------------------- -}

	  if (synchronized) {
	    // save and restore any potential method result value around the unlocking operation
	    save_native_result();
	
	    __ add( __ top_most_monitor(), O1);
	    __ unlock_object(O1);
	
	    restore_native_result();
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 32bit 環境でかつ C2 利用時でない場合には, 不要なので生成しない)
      「32bit 環境では long 値だけは扱いが特殊なので調整しておく」
      (正確に言うと, 32bit 環境では C2 とインタープリタで使うレジスタがずれている.
       C2 は G1 を使うがインタープリタは O0 と O1 を使う.
       ここで, 念のため G1 にも値をコピーしている.)
      ---------------------------------------- -}

	#if defined(COMPILER2) && !defined(_LP64)
	
	  // C2 expects long results in G1 we can't tell if we're returning to interpreted
	  // or compiled so just be safe.
	
	  __ sllx(O0, 32, G1);          // Shift bits into high G1
	  __ srl (O1, 0, O1);           // Zero extend O1
	  __ or3 (O1, G1, G1);          // OR 64 bits into G1
	
	#endif /* COMPILER2 && !_LP64 */
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	  // dispose of return address and remove activation
	#ifdef ASSERT
	  {
	    Label ok;
	    __ cmp(I5_savedSP, FP);
	    __ brx(Assembler::greaterEqualUnsigned, false, Assembler::pt, ok);
	    __ delayed()->nop();
	    __ stop("bad I5_savedSP value");
	    __ should_not_reach_here();
	    __ bind(ok);
	  }
	#endif

  {- -------------------------------------------
  (1) コード生成:
      「Lscratch に入っている result handler を呼び出す」
       (これによって, stack frame から return address と呼び出し元の SP を取得し, 
       SP を呼び出し元の SP(I5_savedSP) に復帰させる, 等の処理を行う)
      ---------------------------------------- -}

	  if (TraceJumps) {
	    // Move target to register that is recordable
	    __ mov(Lscratch, G3_scratch);
	    __ JMP(G3_scratch, 0);
	  } else {
	    __ jmp(Lscratch, 0);
	  }
	  __ delayed()->nop();
	
	
  {- -------------------------------------------
  (1) コード生成: (ただし, JIT コンパイルのためにメソッドの呼び出し回数を数える必要がある場合のみ生成する. 
                 より具体的には, UseCompiler オプションまたは CountCompiledCalls オプションが指定されている場合のみ生成)
      「invocation_counter_overflow ラベルによる飛び先では, 
        InterpreterGenerator::generate_counter_overflow() が生成するコードにより, 
        JIT コンパイラを呼び出す (See: ...#TODO)
        その後, Lcontinue ラベルに戻って実行を再開する.」
      ---------------------------------------- -}

	  if (inc_counter) {
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


