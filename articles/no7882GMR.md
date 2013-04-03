---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/interfaceSupport_windows.hpp

### 名前(function name)
```
static inline void serialize_memory(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::write_memory_serialize_page() を呼び出す.
      
      ただし, ネイティブメソッド内で別の例外ハンドラが設定されているとまずいので, 
      念のため SEH で囲んだ状態で呼び出す.
      (例外ハンドラは os::win32::serialize_fault_filter())
  
      (コメントによると, 
       この処理は性能上クリティカルなパスにあるが, __try/__except は非常に軽い処理なので使って問題ない, 
       とのこと)
      ---------------------------------------- -}

	  // due to chained nature of SEH handlers we have to be sure
	  // that our handler is always last handler before an attempt to write
	  // into serialization page - it can fault if we access this page
	  // right in the middle of protect/unprotect sequence by remote
	  // membar logic.
	  // __try/__except are very lightweight operations (only several
	  // instructions not affecting control flow directly on x86)
	  // so we can use it here, on very time critical path
	  __try {
	    os::write_memory_serialize_page(thread);
	  } __except (os::win32::
	              serialize_fault_filter((_EXCEPTION_POINTERS*)_exception_info()))
	    {}
	}
	
```


