---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// Note: this function shouldn't block if it's called in
// _thread_in_native_trans state (such as from
// check_special_condition_for_native_trans()).
```

### 名前(function name)
```
void JavaThread::check_and_handle_async_exceptions(bool check_unsafe_error) {
```

### 本体部(body)
```
	
	  if (has_last_Java_frame() && has_async_condition()) {
	    // If we are at a polling page safepoint (not a poll return)
	    // then we must defer async exception because live registers
	    // will be clobbered by the exception path. Poll return is
	    // ok because the call we a returning from already collides
	    // with exception handling registers and so there is no issue.
	    // (The exception handling path kills call result registers but
	    //  this is ok since the exception kills the result anyway).
	
	    if (is_at_poll_safepoint()) {
	      // if the code we are returning to has deoptimized we must defer
	      // the exception otherwise live registers get clobbered on the
	      // exception path before deoptimization is able to retrieve them.
	      //
	      RegisterMap map(this, false);
	      frame caller_fr = last_frame().sender(&map);
	      assert(caller_fr.is_compiled_frame(), "what?");
	      if (caller_fr.is_deoptimized_frame()) {
	        if (TraceExceptions) {
	          ResourceMark rm;
	          tty->print_cr("deferred async exception at compiled safepoint");
	        }
	        return;
	      }
	    }
	  }
	
	  JavaThread::AsyncRequests condition = clear_special_runtime_exit_condition();
	  if (condition == _no_async_condition) {
	    // Conditions have changed since has_special_runtime_exit_condition()
	    // was called:
	    // - if we were here only because of an external suspend request,
	    //   then that was taken care of above (or cancelled) so we are done
	    // - if we were here because of another async request, then it has
	    //   been cleared between the has_special_runtime_exit_condition()
	    //   and now so again we are done
	    return;
	  }
	
	  // Check for pending async. exception
	  if (_pending_async_exception != NULL) {
	    // Only overwrite an already pending exception, if it is not a threadDeath.
	    if (!has_pending_exception() || !pending_exception()->is_a(SystemDictionary::ThreadDeath_klass())) {
	
	      // We cannot call Exceptions::_throw(...) here because we cannot block
	      set_pending_exception(_pending_async_exception, __FILE__, __LINE__);
	
	      if (TraceExceptions) {
	        ResourceMark rm;
	        tty->print("Async. exception installed at runtime exit (" INTPTR_FORMAT ")", this);
	        if (has_last_Java_frame() ) {
	          frame f = last_frame();
	          tty->print(" (pc: " INTPTR_FORMAT " sp: " INTPTR_FORMAT " )", f.pc(), f.sp());
	        }
	        tty->print_cr(" of type: %s", instanceKlass::cast(_pending_async_exception->klass())->external_name());
	      }
	      _pending_async_exception = NULL;
	      clear_has_async_exception();
	    }
	  }
	
	  if (check_unsafe_error &&
	      condition == _async_unsafe_access_error && !has_pending_exception()) {
	    condition = _no_async_condition;  // done
	    switch (thread_state()) {
	    case _thread_in_vm:
	      {
	        JavaThread* THREAD = this;
	        THROW_MSG(vmSymbols::java_lang_InternalError(), "a fault occurred in an unsafe memory access operation");
	      }
	    case _thread_in_native:
	      {
	        ThreadInVMfromNative tiv(this);
	        JavaThread* THREAD = this;
	        THROW_MSG(vmSymbols::java_lang_InternalError(), "a fault occurred in an unsafe memory access operation");
	      }
	    case _thread_in_Java:
	      {
	        ThreadInVMfromJava tiv(this);
	        JavaThread* THREAD = this;
	        THROW_MSG(vmSymbols::java_lang_InternalError(), "a fault occurred in a recent unsafe memory access operation in compiled Java code");
	      }
	    default:
	      ShouldNotReachHere();
	    }
	  }
	
	  assert(condition == _no_async_condition || has_pending_exception() ||
	         (!check_unsafe_error && condition == _async_unsafe_access_error),
	         "must have handled the async condition, if no exception");
	}
	
```


