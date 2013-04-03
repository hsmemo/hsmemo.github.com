---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/sharedRuntime.cpp

### 名前(function name)
```
address SharedRuntime::continuation_for_implicit_exception(JavaThread* thread,
                                                           address pc,
                                                           SharedRuntime::ImplicitExceptionKind exception_kind)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address target_pc = NULL;
	
  {- -------------------------------------------
  (1) 例外が発生したアドレスがインタープリタ内を指している場合は, 
      発生した例外に応じて, 以下の適切なハンドラをリターン.
  
      * NullPointerException の場合
        Interpreter::throw_NullPointerException_entry()
      * ArithmeticException の場合
        Interpreter::throw_ArithmeticException_entry()
      * StackOverflowError の場合
        Interpreter::throw_StackOverflowError_entry())
  
      (なお, C++ interpreter の場合は, ここに来ることはない)
      ---------------------------------------- -}

	  if (Interpreter::contains(pc)) {
	#ifdef CC_INTERP
	    // C++ interpreter doesn't throw implicit exceptions
	    ShouldNotReachHere();
	#else
	    switch (exception_kind) {
	      case IMPLICIT_NULL:           return Interpreter::throw_NullPointerException_entry();
	      case IMPLICIT_DIVIDE_BY_ZERO: return Interpreter::throw_ArithmeticException_entry();
	      case STACK_OVERFLOW:          return Interpreter::throw_StackOverflowError_entry();
	      default:                      ShouldNotReachHere();
	    }
	#endif // !CC_INTERP

  {- -------------------------------------------
  (1) 例外が発生したアドレスがインタープリタ内ではない場合, 
      発生した例外に応じて, 以下の適切なハンドラをリターン.
  
      * StackOverflowError の場合
        StubRoutines::throw_StackOverflowError_entry()
        (なおこの場合, Deoptimization::cleanup_deopt_info() による...#TODO)
  
      * NullPointerException の場合
        #TODO (nmethod::continuation_for_implicit_exception()?? それ以外にもある??)
  
      * ArithmeticException の場合
        例外を起こした nmethod に対して
        nmethod::continuation_for_implicit_exception() を呼ぶことで
        ハンドラを取得.
      ---------------------------------------- -}

	  } else {
	    switch (exception_kind) {
	      case STACK_OVERFLOW: {
	        // Stack overflow only occurs upon frame setup; the callee is
	        // going to be unwound. Dispatch to a shared runtime stub
	        // which will cause the StackOverflowError to be fabricated
	        // and processed.
	        // For stack overflow in deoptimization blob, cleanup thread.
	        if (thread->deopt_mark() != NULL) {
	          Deoptimization::cleanup_deopt_info(thread, NULL);
	        }
	        return StubRoutines::throw_StackOverflowError_entry();
	      }
	
	      case IMPLICIT_NULL: {
	        if (VtableStubs::contains(pc)) {
	          // We haven't yet entered the callee frame. Fabricate an
	          // exception and begin dispatching it in the caller. Since
	          // the caller was at a call site, it's safe to destroy all
	          // caller-saved registers, as these entry points do.
	          VtableStub* vt_stub = VtableStubs::stub_containing(pc);
	
	          // If vt_stub is NULL, then return NULL to signal handler to report the SEGV error.
	          if (vt_stub == NULL) return NULL;
	
	          if (vt_stub->is_abstract_method_error(pc)) {
	            assert(!vt_stub->is_vtable_stub(), "should never see AbstractMethodErrors from vtable-type VtableStubs");
	            return StubRoutines::throw_AbstractMethodError_entry();
	          } else {
	            return StubRoutines::throw_NullPointerException_at_call_entry();
	          }
	        } else {
	          CodeBlob* cb = CodeCache::find_blob(pc);
	
	          // If code blob is NULL, then return NULL to signal handler to report the SEGV error.
	          if (cb == NULL) return NULL;
	
	          // Exception happened in CodeCache. Must be either:
	          // 1. Inline-cache check in C2I handler blob,
	          // 2. Inline-cache check in nmethod, or
	          // 3. Implict null exception in nmethod
	
	          if (!cb->is_nmethod()) {
	            guarantee(cb->is_adapter_blob() || cb->is_method_handles_adapter_blob(),
	                      "exception happened outside interpreter, nmethods and vtable stubs (1)");
	            // There is no handler here, so we will simply unwind.
	            return StubRoutines::throw_NullPointerException_at_call_entry();
	          }
	
	          // Otherwise, it's an nmethod.  Consult its exception handlers.
	          nmethod* nm = (nmethod*)cb;
	          if (nm->inlinecache_check_contains(pc)) {
	            // exception happened inside inline-cache check code
	            // => the nmethod is not yet active (i.e., the frame
	            // is not set up yet) => use return address pushed by
	            // caller => don't push another return address
	            return StubRoutines::throw_NullPointerException_at_call_entry();
	          }
	
	#ifndef PRODUCT
	          _implicit_null_throws++;
	#endif
	          target_pc = nm->continuation_for_implicit_exception(pc);
	          // If there's an unexpected fault, target_pc might be NULL,
	          // in which case we want to fall through into the normal
	          // error handling code.
	        }
	
	        break; // fall through
	      }
	
	
	      case IMPLICIT_DIVIDE_BY_ZERO: {
	        nmethod* nm = CodeCache::find_nmethod(pc);
	        guarantee(nm != NULL, "must have containing nmethod for implicit division-by-zero exceptions");
	#ifndef PRODUCT
	        _implicit_div0_throws++;
	#endif
	        target_pc = nm->continuation_for_implicit_exception(pc);
	        // If there's an unexpected fault, target_pc might be NULL,
	        // in which case we want to fall through into the normal
	        // error handling code.
	        break; // fall through
	      }
	
	      default: ShouldNotReachHere();
	    }
	
    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	    assert(exception_kind == IMPLICIT_NULL || exception_kind == IMPLICIT_DIVIDE_BY_ZERO, "wrong implicit exception kind");
	
    {- -------------------------------------------
  (1.1) (デバッグ用の処理)
        ---------------------------------------- -}

	    // for AbortVMOnException flag
	    NOT_PRODUCT(Exceptions::debug_check_abort("java.lang.NullPointerException"));

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (exception_kind == IMPLICIT_NULL) {
	      Events::log("Implicit null exception at " INTPTR_FORMAT " to " INTPTR_FORMAT, pc, target_pc);
	    } else {
	      Events::log("Implicit division by zero exception at " INTPTR_FORMAT " to " INTPTR_FORMAT, pc, target_pc);
	    }

    {- -------------------------------------------
  (1.1) 結果をリターン.
        ---------------------------------------- -}

	    return target_pc;
	  }
	
  {- -------------------------------------------
  (1) (以下は決して到達しないパス)
      ---------------------------------------- -}

	  ShouldNotReachHere();
	  return NULL;
	}
	
```


