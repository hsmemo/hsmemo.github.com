---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void JavaThread::prepare(jobject jni_thread, ThreadPriority prio) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads_lock->owner() == Thread::current(), "must have threads lock");

  {- -------------------------------------------
  (1) (この関数では, java.lang.Thread オブジェクトと C++ レベルでのスレッド(JavaThread)の関連づけを行う.)
      ---------------------------------------- -}

	  // Link Java Thread object <-> C++ Thread
	
	  // Get the C++ thread object (an oop) from the JNI handle (a jthread)
	  // and put it into a new Handle.  The Handle "thread_oop" can then
	  // be used to pass the C++ thread object to other methods.
	
	  // Set the Java level thread object (jthread) field of the
	  // new thread (a JavaThread *) to C++ thread object using the
	  // "thread_oop" handle.
	
	  // Set the thread field (a JavaThread *) of the
	  // oop representing the java_lang_Thread to the new thread (a JavaThread *).
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (thread_oop は, C++ Thread を指す Handle)
      ---------------------------------------- -}

	  Handle thread_oop(Thread::current(),
	                    JNIHandles::resolve_non_null(jni_thread));

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(instanceKlass::cast(thread_oop->klass())->is_linked(),
	    "must be initialized");

  {- -------------------------------------------
  (1) JavaThread::set_thread() を呼んで, 
      JavaThread (C++ Thread) の threadObj フィールドに
      対応する java.lang.Thread オブジェクトをセットしておく.
      ---------------------------------------- -}

	  set_threadObj(thread_oop());

  {- -------------------------------------------
  (1) java_lang_Thread::set_thread() を呼んで, 
      java.lang.Thread オブジェクトの eetop フィールドに
      対応する JavaThread をセットしておく.
  
      (See: [here](no30595cP.html) for details)
      ---------------------------------------- -}

	  java_lang_Thread::set_thread(thread_oop(), this);
	
  {- -------------------------------------------
  (1) Thread::set_priority() を呼んで, このスレッドの優先度を設定する.
  
      なお, 指定する優先度は以下の通り.
      * 引数(prio)で指定されている場合 (= prio が NoPriority ではない場合): 
        prio の値をそのまま使用する.
      * 〃 指定されていない場合 (= prio が NoPriority の場合): 
        処理対象のスレッドに対応する java.lang.Thread オブジェクトから 
        prioirty フィールドの値を取り出して使用する. (See: [here](no3059sSJ.html) for details)
      ---------------------------------------- -}

	  if (prio == NoPriority) {
	    prio = java_lang_Thread::priority(thread_oop());
	    assert(prio != NoPriority, "A valid priority should be present");
	  }
	
	  // Push the Java priority down to the native thread; needs Threads_lock
	  Thread::set_priority(this, prio);
	
  {- -------------------------------------------
  (1) Threads::add() を呼んで, カレントスレッドを Threads::_thread_list に追加する.
  
      (なおコメントによると, 
       Threads::add() を呼び出す際には Threads_lock を取っておく必要がある.
       また, Threads::_thread_list に追加するまではブロックしてはいけない.
       なぜなら, リストに追加するまではこの java_thread は GC の調査対象に入らないから.
       とのこと.)
      ---------------------------------------- -}

	  // Add the new thread to the Threads list and set it in motion.
	  // We must have threads lock in order to call Threads::add.
	  // It is crucial that we do not block before the thread is
	  // added to the Threads list for if a GC happens, then the java_thread oop
	  // will not be visited by GC.
	  Threads::add(this);
	}
	
```


