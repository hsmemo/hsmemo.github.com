---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// Threads_lock NOT held, java_thread not protected by lock
// java_thread - pre-checked
// java_thread - unchecked
// depth - pre-checked as non-negative
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::SetLocalInt(JavaThread* java_thread, jint depth, jint slot, jint value) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // rm object is created to clean up the javaVFrame created in
	  // doit_prologue(), but after doit() is finished with it.
	  ResourceMark rm;
	  jvalue val;

  {- -------------------------------------------
  (1) VM_GetOrSetLocal を呼び出して, 指定された局所変数の値を変更する.
      ---------------------------------------- -}

	  val.i = value;
	  VM_GetOrSetLocal op(java_thread, depth, slot, T_INT, val);
	  VMThread::execute(&op);
	  return op.result();
	} /* end SetLocalInt */
	
```


