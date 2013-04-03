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
// value - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetLocalInstance(JavaThread* java_thread, jint depth, jobject* value_ptr){
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* current_thread = JavaThread::current();
	  // rm object is created to clean up the javaVFrame created in
	  // doit_prologue(), but after doit() is finished with it.
	  ResourceMark rm(current_thread);
	
  {- -------------------------------------------
  (1) VM_GetReceiver を呼び出して指定された局所変数の値を取得する.
      取得に成功したら, 結果を value_ptr 引数が指している箇所に書き込んでリターン.
      (取得に失敗したら, VM_GetOrSetLocal のエラー値をそのままリターンするだけ)
      ---------------------------------------- -}

	  VM_GetReceiver op(java_thread, current_thread, depth);
	  VMThread::execute(&op);
	  jvmtiError err = op.result();
	  if (err != JVMTI_ERROR_NONE) {
	    return err;
	  } else {
	    *value_ptr = op.value().l;
	    return JVMTI_ERROR_NONE;
	  }
	} /* end GetLocalInstance */
	
```


