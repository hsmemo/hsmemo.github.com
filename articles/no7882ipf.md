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
	  push(rax);
	  push(rbx);
	  push(c_rarg3);
	  push(c_rarg2);
	  test_method_data_pointer(c_rarg3, verify_continue); // If mdp is zero, continue
	  get_method(rbx);
	
	  // If the mdp is valid, it will point to a DataLayout header which is
	  // consistent with the bcp.  The converse is highly probable also.
	  load_unsigned_short(c_rarg2,
	                      Address(c_rarg3, in_bytes(DataLayout::bci_offset())));
	  addptr(c_rarg2, Address(rbx, methodOopDesc::const_offset()));
	  lea(c_rarg2, Address(c_rarg2, constMethodOopDesc::codes_offset()));
	  cmpptr(c_rarg2, r13);
	  jcc(Assembler::equal, verify_continue);
	  // rbx: method
	  // r13: bcp
	  // c_rarg3: mdp
	  call_VM_leaf(CAST_FROM_FN_PTR(address, InterpreterRuntime::verify_mdp),
	               rbx, r13, c_rarg3);
	  bind(verify_continue);
	  pop(c_rarg2);
	  pop(c_rarg3);
	  pop(rbx);
	  pop(rax);
	#endif // ASSERT
	}
	
```


