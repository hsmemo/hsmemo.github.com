---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/stubGenerator_x86_32.cpp

### 名前(function name)
```
  void generate_all() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    // Generates all stubs and initializes the entry points
	
	    // These entry points require SharedInfo::stack0 to be set up in non-core builds
	    // and need to be relocatable, so they each fabricate a RuntimeStub internally.
	    StubRoutines::_throw_AbstractMethodError_entry         = generate_throw_exception("AbstractMethodError throw_exception",          CAST_FROM_FN_PTR(address, SharedRuntime::throw_AbstractMethodError),  false);
	    StubRoutines::_throw_IncompatibleClassChangeError_entry= generate_throw_exception("IncompatibleClassChangeError throw_exception", CAST_FROM_FN_PTR(address, SharedRuntime::throw_IncompatibleClassChangeError),  false);
	    StubRoutines::_throw_ArithmeticException_entry         = generate_throw_exception("ArithmeticException throw_exception",          CAST_FROM_FN_PTR(address, SharedRuntime::throw_ArithmeticException),  true);
	    StubRoutines::_throw_NullPointerException_entry        = generate_throw_exception("NullPointerException throw_exception",         CAST_FROM_FN_PTR(address, SharedRuntime::throw_NullPointerException), true);
	    StubRoutines::_throw_NullPointerException_at_call_entry= generate_throw_exception("NullPointerException at call throw_exception", CAST_FROM_FN_PTR(address, SharedRuntime::throw_NullPointerException_at_call), false);
	    StubRoutines::_throw_StackOverflowError_entry          = generate_throw_exception("StackOverflowError throw_exception",           CAST_FROM_FN_PTR(address, SharedRuntime::throw_StackOverflowError),   false);
	
	    //------------------------------------------------------------------------------------------------------------------------
	    // entry points that are platform specific
	
	    // support for verify_oop (must happen after universe_init)
	    StubRoutines::_verify_oop_subroutine_entry     = generate_verify_oop();
	
	    // arraycopy stubs used by compilers
	    generate_arraycopy_stubs();
	
	    generate_math_stubs();
	  }
	
```


