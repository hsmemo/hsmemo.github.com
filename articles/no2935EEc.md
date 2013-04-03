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
// thread_count - pre-checked to be greater than or equal to 0
// thread_list - pre-checked for NULL
// max_frame_count - pre-checked to be greater than or equal to 0
// stack_info_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetThreadListStackTraces(jint thread_count, const jthread* thread_list, jint max_frame_count, jvmtiStackInfo** stack_info_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;

  {- -------------------------------------------
  (1) VM_GetThreadListStackTraces でスタックトレース情報を取得する.
      ---------------------------------------- -}

	  // JVMTI get stack traces at safepoint.
	  VM_GetThreadListStackTraces op(this, thread_count, thread_list, max_frame_count);
	  VMThread::execute(&op);

  {- -------------------------------------------
  (1) 取得した結果を返値にセットする.
      取得処理が成功していれば, 引数で渡されたポインタ(stack_info_ptr)にも結果をセットしておく.
      ---------------------------------------- -}

	  err = op.result();
	  if (err == JVMTI_ERROR_NONE) {
	    *stack_info_ptr = op.stack_info();
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return err;
	} /* end GetThreadListStackTraces */
	
```


