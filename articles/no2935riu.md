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
VM_GetAllStackTraces::doit() {
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
	  _final_thread_count = 0;

  {- -------------------------------------------
  (1) 全ての JavaThread に対して VM_GetMultipleStackTraces::fill_frames() を呼び出し, 
      処理対象として登録しておく.
  
      (ただし, 既に死んでいる JavaThread については登録しない. 
       また HotSpot が内部的に使用するスレッド(= is_hidden_from_external_view() が true) についても登録しない)
      ---------------------------------------- -}

	  for (JavaThread *jt = Threads::first(); jt != NULL; jt = jt->next()) {
	    oop thread_oop = jt->threadObj();
	    if (thread_oop != NULL &&
	        !jt->is_exiting() &&
	        java_lang_Thread::is_alive(thread_oop) &&
	        !jt->is_hidden_from_external_view()) {
	      ++_final_thread_count;
	      // Handle block of the calling thread is used to create local refs.
	      fill_frames((jthread)JNIHandles::make_local(_calling_thread, thread_oop),
	                  jt, thread_oop);
	    }
	  }

  {- -------------------------------------------
  (1) VM_GetMultipleStackTraces::allocate_and_fill_stacks() を呼び出して, 
      スタックトレース情報を取得する.
      ---------------------------------------- -}

	  allocate_and_fill_stacks(_final_thread_count);
	}
	
```


