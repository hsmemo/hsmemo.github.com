---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/stubGenerator_x86_64.cpp
### 説明(description)

```
  // Initialization
```

### 名前(function name)
```
  void generate_initial() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Generates all stubs and initializes the entry points
	
	    // This platform-specific stub is needed by generate_call_stub()
	    StubRoutines::x86::_mxcsr_std        = generate_fp_mask("mxcsr_std",        0x0000000000001F80);
	
	    // entry points that exist in all platforms Note: This is code
	    // that could be shared among different platforms - however the
	    // benefit seems to be smaller than the disadvantage of having a
	    // much more complicated generator structure. See also comment in
	    // stubRoutines.hpp.
	
	    StubRoutines::_forward_exception_entry = generate_forward_exception();
	
	    StubRoutines::_call_stub_entry =
	      generate_call_stub(StubRoutines::_call_stub_return_address);
	
	    // is referenced by megamorphic call
	    StubRoutines::_catch_exception_entry = generate_catch_exception();
	
	    // atomic calls
	    StubRoutines::_atomic_xchg_entry         = generate_atomic_xchg();
	    StubRoutines::_atomic_xchg_ptr_entry     = generate_atomic_xchg_ptr();
	    StubRoutines::_atomic_cmpxchg_entry      = generate_atomic_cmpxchg();
	    StubRoutines::_atomic_cmpxchg_long_entry = generate_atomic_cmpxchg_long();
	    StubRoutines::_atomic_add_entry          = generate_atomic_add();
	    StubRoutines::_atomic_add_ptr_entry      = generate_atomic_add_ptr();
	    StubRoutines::_fence_entry               = generate_orderaccess_fence();
	
	    StubRoutines::_handler_for_unsafe_access_entry =
	      generate_handler_for_unsafe_access();
	
	    // platform dependent
	    StubRoutines::x86::_get_previous_fp_entry = generate_get_previous_fp();
	
	    StubRoutines::x86::_verify_mxcsr_entry    = generate_verify_mxcsr();
	
	    // Build this early so it's available for the interpreter.  Stub
	    // expects the required and actual types as register arguments in
	    // j_rarg0 and j_rarg1 respectively.
	    StubRoutines::_throw_WrongMethodTypeException_entry =
	      generate_throw_exception("WrongMethodTypeException throw_exception",
	                               CAST_FROM_FN_PTR(address, SharedRuntime::throw_WrongMethodTypeException),
	                               false, rax, rcx);
	  }
	
```


