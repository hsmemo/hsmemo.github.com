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
void InterpreterMacroAssembler::call_VM_leaf_base(address entry_point,
                                                  int number_of_arguments) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なおコメントによると, r13(bcp) と r14(locals) は callee save であるうえ
       leaf だから GC 等も生じないので, 待避復帰する必要は無いとのこと.
       というか, むしろしてはいけないとのこと.
       caller が一時的に値を待避して別のことに使っているかもしれないので, 
       ここでさらに待避すると元の値を壊す恐れがある.)
      ---------------------------------------- -}

	  // interpreter specific
	  //
	  // Note: No need to save/restore bcp & locals (r13 & r14) pointer
	  //       since these are callee saved registers and no blocking/
	  //       GC can happen in leaf calls.
	  // Further Note: DO NOT save/restore bcp/locals. If a caller has
	  // already saved them so that it can use esi/edi as temporaries
	  // then a save/restore here will DESTROY the copy the caller
	  // saved! There used to be a save_bcp() that only happened in
	  // the ASSERT path (no restore_bcp). Which caused bizarre failures
	  // when jvm built with ASSERTs.
	#ifdef ASSERT
	  {
	    Label L;
	    cmpptr(Address(rbp, frame::interpreter_frame_last_sp_offset * wordSize), (int32_t)NULL_WORD);
	    jcc(Assembler::equal, L);
	    stop("InterpreterMacroAssembler::call_VM_leaf_base:"
	         " last_sp != NULL");
	    bind(L);
	  }
	#endif

  {- -------------------------------------------
  (1) コード生成:
      「MacroAssembler::call_VM_leaf_base() が生成するコードにより, 
        引数で指定されたエントリポイントを呼び出す.」
      ---------------------------------------- -}

	  // super call
	  MacroAssembler::call_VM_leaf_base(entry_point, number_of_arguments);

  {- -------------------------------------------
  (1) (ここには r13/r14 の値をチェックする ASSERT があったが, 上記の理由から今はないとのこと)
      ---------------------------------------- -}

	  // interpreter specific
	  // Used to ASSERT that r13/r14 were equal to frame's bcp/locals
	  // but since they may not have been saved (and we don't want to
	  // save thme here (see note above) the assert is invalid.
	}
	
```


