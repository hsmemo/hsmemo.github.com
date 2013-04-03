---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp

### 名前(function name)
```
void InterpreterMacroAssembler::check_and_handle_earlyret(Register scratch_reg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (JvmtiExport::can_force_early_return() が true でない場合は, (何もする必要が無いので) 以降の処理は全て省略)
      ---------------------------------------- -}

	  if (JvmtiExport::can_force_early_return()) {

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    Label L;

  {- -------------------------------------------
  (1) コード生成:
      「もし カレントスレッドが JvmtiThreadState を持っており (= JavaThread::jvmti_thread_state() が NULL ではなく),
        かつその JvmtiThreadState::_earlyret_state フィールドの値が JvmtiThreadState::earlyret_pending であれば,
        Interpreter::remove_activation_early_entry() が返すアドレスにジャンプする.」
      ---------------------------------------- -}

	    Register thr_state = G3_scratch;
	    ld_ptr(G2_thread, JavaThread::jvmti_thread_state_offset(), thr_state);
	    tst(thr_state);
	    br(zero, false, pt, L); // if (thread->jvmti_thread_state() == NULL) exit;
	    delayed()->nop();
	
	    // Initiate earlyret handling only if it is not already being processed.
	    // If the flag has the earlyret_processing bit set, it means that this code
	    // is called *during* earlyret handling - we don't want to reenter.
	    ld(thr_state, JvmtiThreadState::earlyret_state_offset(), G4_scratch);
	    cmp(G4_scratch, JvmtiThreadState::earlyret_pending);
	    br(Assembler::notEqual, false, pt, L);
	    delayed()->nop();
	
	    // Call Interpreter::remove_activation_early_entry() to get the address of the
	    // same-named entrypoint in the generated interpreter code
	    ld(thr_state, JvmtiThreadState::earlyret_tos_offset(), Otos_l1);
	    call_VM_leaf(noreg, CAST_FROM_FN_PTR(address, Interpreter::remove_activation_early_entry), Otos_l1);
	
	    // Jump to Interpreter::_remove_activation_early_entry
	    jmpl(O0, G0, G0);
	    delayed()->nop();
	    bind(L);
	  }
	}
	
```


