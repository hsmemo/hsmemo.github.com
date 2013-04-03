---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/windows_x86/vm/os_windows_x86.cpp
### 説明(description)
// Install a win32 structured exception handler around thread.


### 名前(function name)
```
void os::os_exception_wrapper(java_call_t f, JavaValue* value, methodHandle* method, JavaCallArguments* args, Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SEH で囲んだ状態で, 引数で渡された関数を呼び出す.
      (なお, 例外ハンドラは topLevelExceptionFilter.
       See: [here](no30592eE.html) for details)
      ---------------------------------------- -}

	  __try {
	
	#ifndef AMD64
	    // We store the current thread in this wrapperthread location
	    // and determine how far away this address is from the structured
	    // execption pointer that FS:[0] points to.  This get_thread
	    // code can then get the thread pointer via FS.
	    //
	    // Warning:  This routine must NEVER be inlined since we'd end up with
	    //           multiple offsets.
	    //
	    volatile Thread* wrapperthread = thread;
	
	    if ( ThreadLocalStorage::get_thread_ptr_offset() == 0 ) {
	      int thread_ptr_offset;
	      __asm {
	        lea eax, dword ptr wrapperthread;
	        sub eax, dword ptr FS:[0H];
	        mov thread_ptr_offset, eax
	      };
	      ThreadLocalStorage::set_thread_ptr_offset(thread_ptr_offset);
	    }
	#ifdef ASSERT
	    // Verify that the offset hasn't changed since we initally captured
	    // it. This might happen if we accidentally ended up with an
	    // inlined version of this routine.
	    else {
	      int test_thread_ptr_offset;
	      __asm {
	        lea eax, dword ptr wrapperthread;
	        sub eax, dword ptr FS:[0H];
	        mov test_thread_ptr_offset, eax
	      };
	      assert(test_thread_ptr_offset == ThreadLocalStorage::get_thread_ptr_offset(),
	             "thread pointer offset from SEH changed");
	    }
	#endif // ASSERT
	#endif // !AMD64
	
	    f(value, method, args, thread);
	  } __except(topLevelExceptionFilter((_EXCEPTION_POINTERS*)_exception_info())) {
	      // Nothing to do.
	  }
	}
	
```


