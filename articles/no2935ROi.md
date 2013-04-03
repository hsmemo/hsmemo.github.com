---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp

### 名前(function name)
```
void
VM_GetThreadListStackTraces::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(SafepointSynchronize::is_at_safepoint(), "must be at safepoint");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;

  {- -------------------------------------------
  (1) コンストラクタ引数で指定された全ての JavaThread に対して 
      VM_GetMultipleStackTraces::fill_frames() を呼び出し, 
      処理対象として登録しておく.
  
      (なお, 引数に不正なポインタや Thread オブジェクトで無いものが混じっていた場合はこの時点でリターン)
      ---------------------------------------- -}

	  for (int i = 0; i < _thread_count; ++i) {
	    jthread jt = _thread_list[i];
	    oop thread_oop = JNIHandles::resolve_external_guard(jt);
	    if (thread_oop == NULL || !thread_oop->is_a(SystemDictionary::Thread_klass())) {
	      set_result(JVMTI_ERROR_INVALID_THREAD);
	      return;
	    }
	    fill_frames(jt, java_lang_Thread::thread(thread_oop), thread_oop);
	  }

  {- -------------------------------------------
  (1) VM_GetMultipleStackTraces::allocate_and_fill_stacks() を呼び出して, 
      スタックトレース情報を取得する.
      ---------------------------------------- -}

	  allocate_and_fill_stacks(_thread_count);
	}
	
```


