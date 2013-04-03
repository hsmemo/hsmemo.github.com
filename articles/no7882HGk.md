---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
#define JVM_ENTRY_NO_ENV(result_type, header)                        \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (関数定義の関数名および型宣言部が生成される)
      ---------------------------------------- -}

	extern "C" {                                                         \
	  result_type JNICALL header {                                       \

  {- -------------------------------------------
  (1) (変数宣言など)
      (See: ThreadInVMfromNative)
      ---------------------------------------- -}

	    JavaThread* thread = (JavaThread*)ThreadLocalStorage::thread();  \
	    ThreadInVMfromNative __tiv(thread);                              \
	    debug_only(VMNativeEntryWrapper __vew;)                          \

  {- -------------------------------------------
  (1) __ENTRY() マクロのコードが展開される
      ---------------------------------------- -}

	    __ENTRY(result_type, header, thread)
	
```


