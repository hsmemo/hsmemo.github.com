---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
ThreadsListEnumerator::ThreadsListEnumerator(Thread* cur_thread,
                                             bool include_jvmti_agent_threads,
                                             bool include_jni_attaching_threads) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(cur_thread == Thread::current(), "Check current thread");
	
  {- -------------------------------------------
  (1) 処理結果は GrowableArray という形にして
      _threads_array フィールドに入れておく必要があるため, 
      全スレッド分の大きさを持った GrowableArray を確保する.
      (全スレッド数は ThreadService::get_live_thread_count() で取得)
      ---------------------------------------- -}

	  int init_size = ThreadService::get_live_thread_count();
	  _threads_array = new GrowableArray<instanceHandle>(init_size);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  MutexLockerEx ml(Threads_lock);
	
  {- -------------------------------------------
  (1) 全ての JavaThread を辿り, _threads_array フィールドに格納していく.
  
      ただし, 以下の JavaThread については, 
      引数によって結果に含まれたり含まれなかったりする.
      * JVMTI の RunAgentThread() で使われるスレッド
        引数の include_jvmti_agent_threads が true の場合のみ結果に含める (false の場合は無視する)
        (See: JvmtiAgentThread)
      * JNI の AttachCurrentThread*() で作られたスレッド
        引数の include_jni_attaching_threads が true の場合のみ結果に含める (false の場合は無視する)
        (See: jni_AttachCurrentThread(), jni_AttachCurrentThreadAsDaemon())
      ---------------------------------------- -}

	  for (JavaThread* jt = Threads::first(); jt != NULL; jt = jt->next()) {
	    // skips JavaThreads in the process of exiting
	    // and also skips VM internal JavaThreads
	    // Threads in _thread_new or _thread_new_trans state are included.
	    // i.e. threads have been started but not yet running.
	    if (jt->threadObj() == NULL   ||
	        jt->is_exiting() ||
	        !java_lang_Thread::is_alive(jt->threadObj())   ||
	        jt->is_hidden_from_external_view()) {
	      continue;
	    }
	
	    // skip agent threads
	    if (!include_jvmti_agent_threads && jt->is_jvmti_agent_thread()) {
	      continue;
	    }
	
	    // skip jni threads in the process of attaching
	    if (!include_jni_attaching_threads && jt->is_attaching()) {
	      continue;
	    }
	
	    instanceHandle h(cur_thread, (instanceOop) jt->threadObj());
	    _threads_array->append(h);
	  }
	}
	
```


