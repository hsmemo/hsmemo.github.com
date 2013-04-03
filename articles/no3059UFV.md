---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp

### 名前(function name)
```
void InterpreterMacroAssembler::call_VM_base(Register oop_result,
                                             Register java_thread,
                                             Register last_java_sp,
                                             address  entry_point,
                                             int      number_of_arguments,
                                             bool     check_exceptions) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM_base() が生成するコードによって呼び出しを行う.
    
       (なお, 以下の関数が生成するコードにより, 
        呼び出しの前後で r13(bcp) や r14(locals)の待避復帰も行っている.
  
        InterpreterMacroAssembler::save_bcp(), 
        InterpreterMacroAssembler::restore_bcp(), 
        InterpreterMacroAssembler::restore_locals())
  
      (なおコメントによると, 
       restore_locals() は省略できるかもしれないが, 元が重いからどのみち差なんか出ないよ, 
       とのこと)
      ---------------------------------------- -}

	  // interpreter specific
	  //
	  // Note: Could avoid restoring locals ptr (callee saved) - however doesn't
	  //       really make a difference for these runtime calls, since they are
	  //       slow anyway. Btw., bcp must be saved/restored since it may change
	  //       due to GC.
	  // assert(java_thread == noreg , "not expecting a precomputed java thread");
	  save_bcp();
	#ifdef ASSERT
	  {
	    Label L;
	    cmpptr(Address(rbp, frame::interpreter_frame_last_sp_offset * wordSize), (int32_t)NULL_WORD);
	    jcc(Assembler::equal, L);
	    stop("InterpreterMacroAssembler::call_VM_leaf_base:"
	         " last_sp != NULL");
	    bind(L);
	  }
	#endif /* ASSERT */
	  // super call
	  MacroAssembler::call_VM_base(oop_result, noreg, last_java_sp,
	                               entry_point, number_of_arguments,
	                               check_exceptions);
	  // interpreter specific
	  restore_bcp();
	  restore_locals();
	}
	
```


