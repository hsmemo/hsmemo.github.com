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
// Threads_lock NOT held
// thread - NOT pre-checked
// proc - pre-checked for NULL
// arg - NULL is a valid value, must be checked
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::RunAgentThread(jthread thread, jvmtiStartFunction proc, const void* arg, jint priority) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし thread 引数の値がおかしければ, ここでリターン(JVMTI_ERROR_INVALID_THREAD)
      ---------------------------------------- -}

	  oop thread_oop = JNIHandles::resolve_external_guard(thread);
	  if (thread_oop == NULL || !thread_oop->is_a(SystemDictionary::Thread_klass())) {
	    return JVMTI_ERROR_INVALID_THREAD;
	  }

  {- -------------------------------------------
  (1) もし priority 引数の値がおかしければ, ここでリターン(JVMTI_ERROR_INVALID_PRIORITY)
      ---------------------------------------- -}

	  if (priority < JVMTI_THREAD_MIN_PRIORITY || priority > JVMTI_THREAD_MAX_PRIORITY) {
	    return JVMTI_ERROR_INVALID_PRIORITY;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  //Thread-self
	  JavaThread* current_thread = JavaThread::current();
	
	  Handle thread_hndl(current_thread, thread_oop);
	  {

  {- -------------------------------------------
  (1) (以降の処理は Threads_lock で排他した状態で行う)
      ---------------------------------------- -}

	    MutexLocker mu(Threads_lock); // grab Threads_lock
	
  {- -------------------------------------------
  (1) 新しい JvmtiAgentThread オブジェクトを生成する.
      (もしメモリ確保や OSThread の生成に失敗したら, ここでリターン(JVMTI_ERROR_OUT_OF_MEMORY))
      ---------------------------------------- -}

	    JvmtiAgentThread *new_thread = new JvmtiAgentThread(this, proc, arg);
	
	    // At this point it may be possible that no osthread was created for the
	    // JavaThread due to lack of memory.
	    if (new_thread == NULL || new_thread->osthread() == NULL) {
	      if (new_thread) delete new_thread;
	      return JVMTI_ERROR_OUT_OF_MEMORY;
	    }
	
  {- -------------------------------------------
  (1) java_lang_Thread::set_thread() を呼んで, 
      thread 引数で指定された java.lang.Thread オブジェクトの eetop フィールドに
      生成した JvmtiAgentThread オブジェクトをセットしておく.
  
      (See: [here](no30595cP.html) for details)
      ---------------------------------------- -}

	    java_lang_Thread::set_thread(thread_hndl(), new_thread);

  {- -------------------------------------------
  (1) java_lang_Thread::set_priority() を呼んで, 処理対象のスレッドの優先度を priority 引数の値に設定する.
      ---------------------------------------- -}

	    java_lang_Thread::set_priority(thread_hndl(), (ThreadPriority)priority);

  {- -------------------------------------------
  (1) java_lang_Thread::set_daemon() を呼んで, 処理対象のスレッドをデーモンスレッドにする
      ---------------------------------------- -}

	    java_lang_Thread::set_daemon(thread_hndl());
	
  {- -------------------------------------------
  (1) JavaThread::set_thread() を呼んで, 
      生成した JvmtiAgentThread の threadObj フィールドに
      処理対象の java.lang.Thread オブジェクトをセットしておく.
      ---------------------------------------- -}

	    new_thread->set_threadObj(thread_hndl());

  {- -------------------------------------------
  (1) Threads::add() を呼んで, 生成した JvmtiAgentThread を Threads::_thread_list に追加する.
      ---------------------------------------- -}

	    Threads::add(new_thread);

  {- -------------------------------------------
  (1) 生成した JvmtiAgentThread の実行を開始させる.
      ---------------------------------------- -}

	    Thread::start(new_thread);

  {- -------------------------------------------
  (1) (Threads_lock で排他している区間はここまで)
      ---------------------------------------- -}

	  } // unlock Threads_lock
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end RunAgentThread */
	
```


