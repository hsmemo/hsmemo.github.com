---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
LONG WINAPI os::win32::serialize_fault_filter(struct _EXCEPTION_POINTERS* e) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし例外が EXCEPTION_ACCESS_VIOLATION であり, かつ
      そのアドレスが serialize page であれば (= os::is_memory_serialize_page() が true を返せば), 
      EXCEPTION_CONTINUE_EXECUTION をリターンする.
    
      それ以外の場合には, EXCEPTION_CONTINUE_SEARCH をリターンする.
  
      
      (この場合は, 何故 os::block_on_serialize_page_trap() で待機していない?? #TODO)
      ---------------------------------------- -}

	  DWORD exception_code = e->ExceptionRecord->ExceptionCode;
	
	  if ( exception_code == EXCEPTION_ACCESS_VIOLATION ) {
	    JavaThread* thread = (JavaThread*)ThreadLocalStorage::get_thread_slow();
	    PEXCEPTION_RECORD exceptionRecord = e->ExceptionRecord;
	    address addr = (address) exceptionRecord->ExceptionInformation[1];
	
	    if (os::is_memory_serialize_page(thread, addr))
	      return EXCEPTION_CONTINUE_EXECUTION;
	  }
	
	  return EXCEPTION_CONTINUE_SEARCH;
	}
	
```


