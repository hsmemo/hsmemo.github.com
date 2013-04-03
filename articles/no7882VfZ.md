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
void InterpreterMacroAssembler::verify_method_data_pointer() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ProfileInterpreter, "must be profiling interpreter");

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  Label verify_continue;
	  test_method_data_pointer(verify_continue);
	
	  // If the mdp is valid, it will point to a DataLayout header which is
	  // consistent with the bcp.  The converse is highly probable also.
	  lduh(ImethodDataPtr, in_bytes(DataLayout::bci_offset()), G3_scratch);
	  ld_ptr(Lmethod, methodOopDesc::const_offset(), O5);
	  add(G3_scratch, in_bytes(constMethodOopDesc::codes_offset()), G3_scratch);
	  add(G3_scratch, O5, G3_scratch);
	  cmp(Lbcp, G3_scratch);
	  brx(Assembler::equal, false, Assembler::pt, verify_continue);
	
	  Register temp_reg = O5;
	  delayed()->mov(ImethodDataPtr, temp_reg);
	  // %%% should use call_VM_leaf here?
	  //call_VM_leaf(noreg, ..., Lmethod, Lbcp, ImethodDataPtr);
	  save_frame_and_mov(sizeof(jdouble) / wordSize, Lmethod, O0, Lbcp, O1);
	  Address d_save(FP, -sizeof(jdouble) + STACK_BIAS);
	  stf(FloatRegisterImpl::D, Ftos_d, d_save);
	  mov(temp_reg->after_save(), O2);
	  save_thread(L7_thread_cache);
	  call(CAST_FROM_FN_PTR(address, InterpreterRuntime::verify_mdp), relocInfo::none);
	  delayed()->nop();
	  restore_thread(L7_thread_cache);
	  ldf(FloatRegisterImpl::D, d_save, Ftos_d);
	  restore();
	  bind(verify_continue);
	#endif // ASSERT
	}
	
```


