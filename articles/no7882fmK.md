---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp
### 説明(description)

```
// Ensure that the VMNativeEntryWrapper constructor, which can cause
// a GC, is called outside the NoHandleMark (set via __QUICK_ENTRY).
```

### 名前(function name)
```
#define JNI_QUICK_ENTRY(result_type, header)                         \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (関数定義の関数名および型宣言部が生成される)
      ---------------------------------------- -}

	extern "C" {                                                         \
	  result_type JNICALL header {                                \

  {- -------------------------------------------
  (1) (変数宣言など)
      (See: ThreadInVMfromNative)
      ---------------------------------------- -}

	    JavaThread* thread=JavaThread::thread_from_jni_environment(env); \
	    assert( !VerifyJNIEnvThread || (thread == Thread::current()), "JNIEnv is only valid in same thread"); \
	    ThreadInVMfromNative __tiv(thread);                              \
	    debug_only(VMNativeEntryWrapper __vew;)                          \

  {- -------------------------------------------
  (1) __QUICK_ENTRY() マクロのコードが展開される
      ---------------------------------------- -}

	    __QUICK_ENTRY(result_type, header, thread)
	
```


