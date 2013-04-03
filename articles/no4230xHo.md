---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vm_operations.cpp

### 名前(function name)
```
void VM_Exit::doit() {
```

### 本体部(body)
```
	  CompileBroker::set_should_block();
	
	  // Wait for a short period for threads in native to block. Any thread
	  // still executing native code after the wait will be stopped at
	  // native==>Java/VM barriers.
	  // Among 16276 JCK tests, 94% of them come here without any threads still
	  // running in native; the other 6% are quiescent within 250ms (Ultra 80).
	  wait_for_threads_in_native_to_block();
	
	  set_vm_exited();
	
	  // cleanup globals resources before exiting. exit_globals() currently
	  // cleans up outputStream resources and PerfMemory resources.
	  exit_globals();
	
	  // Check for exit hook
	  exit_hook_t exit_hook = Arguments::exit_hook();
	  if (exit_hook != NULL) {
	    // exit hook should exit.
	    exit_hook(_exit_code);
	    // ... but if it didn't, we must do it here
	    vm_direct_exit(_exit_code);
	  } else {
	    vm_direct_exit(_exit_code);
	  }
	
```


