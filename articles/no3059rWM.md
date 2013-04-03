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
void InterpreterMacroAssembler::prepare_to_jump_from_interpreted() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SP の値を r13 レジスタに待避する.
      ---------------------------------------- -}

	  // set sender sp
	  lea(r13, Address(rsp, wordSize));

  {- -------------------------------------------
  (1) SP の値をスタックフレーム上にも待避しておく (待避位置は frame::interpreter_frame_last_sp_offset)
      ---------------------------------------- -}

	  // record last_sp
	  movptr(Address(rbp, frame::interpreter_frame_last_sp_offset * wordSize), r13);
	}
	
```


