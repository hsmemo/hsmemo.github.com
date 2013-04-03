---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
JVM_ENTRY(void, JVM_StartThread(JNIEnv* env, jobject jthread))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_StartThread");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread *native_thread = NULL;
	
	  // We cannot hold the Threads_lock when we throw an exception,
	  // due to rank ordering issues. Example:  we might need to grab the
	  // Heap_lock while we construct the exception.
	  bool throw_illegal_thread_state = false;
	
  {- -------------------------------------------
  (1) 新しい JavaThread オブジェクトを生成し, JavaThread::prepare() で初期化を行う.
      (ただし, 対象の java.lang.Thread オブジェクトの状態がおかしい場合は, どちらも行わない)
      (また, JavaThread オブジェクトの生成に失敗した場合は, JavaThread::prepare() の呼び出しは行わない)
      ---------------------------------------- -}

	  // We must release the Threads_lock before we can post a jvmti event
	  // in Thread::start.
	  {
	    // Ensure that the C++ Thread and OSThread structures aren't freed before
	    // we operate.
	    MutexLocker mu(Threads_lock);
	
	    // Since JDK 5 the java.lang.Thread threadStatus is used to prevent
	    // re-starting an already started thread, so we should usually find
	    // that the JavaThread is null. However for a JNI attached thread
	    // there is a small window between the Thread object being created
	    // (with its JavaThread set) and the update to its threadStatus, so we
	    // have to check for this
	    if (java_lang_Thread::thread(JNIHandles::resolve_non_null(jthread)) != NULL) {
	      throw_illegal_thread_state = true;
	    } else {
	      // We could also check the stillborn flag to see if this thread was already stopped, but
	      // for historical reasons we let the thread detect that itself when it starts running
	
	      jlong size =
	             java_lang_Thread::stackSize(JNIHandles::resolve_non_null(jthread));
	      // Allocate the C++ Thread structure and create the native thread.  The
	      // stack size retrieved from java is signed, but the constructor takes
	      // size_t (an unsigned type), so avoid passing negative values which would
	      // result in really large stacks.
	      size_t sz = size > 0 ? (size_t) size : 0;
	      native_thread = new JavaThread(&thread_entry, sz);
	
	      // At this point it may be possible that no osthread was created for the
	      // JavaThread due to lack of memory. Check for this situation and throw
	      // an exception if necessary. Eventually we may want to change this so
	      // that we only grab the lock if the thread was created successfully -
	      // then we can also do this check and throw the exception in the
	      // JavaThread constructor.
	      if (native_thread->osthread() != NULL) {
	        // Note: the current thread is not being used within "prepare".
	        native_thread->prepare(jthread);
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 対象の java.lang.Thread オブジェクトの状態がおかしい場合は, 
      IllegalThreadStateException を出す (この場合はここで終了).
      ---------------------------------------- -}

	  if (throw_illegal_thread_state) {
	    THROW(vmSymbols::java_lang_IllegalThreadStateException());
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(native_thread != NULL, "Starting null thread?");
	
  {- -------------------------------------------
  (1) JavaThread オブジェクトの生成に失敗していた場合は, OutOfMemoryError を出す.
    
      ついでに, (JVMTI のフック点)でもある.
      (See: JvmtiExport::post_resource_exhausted())
      ---------------------------------------- -}

	  if (native_thread->osthread() == NULL) {
	    // No one should hold a reference to the 'native_thread'.
	    delete native_thread;
	    if (JvmtiExport::should_post_resource_exhausted()) {
	      JvmtiExport::post_resource_exhausted(
	        JVMTI_RESOURCE_EXHAUSTED_OOM_ERROR | JVMTI_RESOURCE_EXHAUSTED_THREADS,
	        "unable to create new native thread");
	    }
	    THROW_MSG(vmSymbols::java_lang_OutOfMemoryError(),
	              "unable to create new native thread");
	  }
	
  {- -------------------------------------------
  (1) Thread::start() を呼んで, 生成した JavaThread を開始させる.
      ---------------------------------------- -}

	  Thread::start(native_thread);
	
	JVM_END
	
```


