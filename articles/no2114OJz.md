---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/unsafe.cpp

### 名前(function name)
```
UNSAFE_ENTRY(void, Unsafe_Unpark(JNIEnv *env, jobject unsafe, jobject jthread))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (現状では何もしない) (See: UnsafeWrapper)
      ---------------------------------------- -}

	  UnsafeWrapper("Unsafe_Unpark");

  {- -------------------------------------------
  (1) 処理対象のスレッドの parker フィールドにある Parker オブジェクトを取得する.
  
      (なお, 初回に java_lang_Thread::set_park_event() でメモイズしておき, 
       二回目以降は java_lang_Thread::park_event() で高速に取り出す, という方式を取っている.
       これにより, 二回目以降は Threads_lock を取らずに取得が可能.)
      ---------------------------------------- -}

	  Parker* p = NULL;
	  if (jthread != NULL) {
	    oop java_thread = JNIHandles::resolve_non_null(jthread);
	    if (java_thread != NULL) {
	      jlong lp = java_lang_Thread::park_event(java_thread);
	      if (lp != 0) {
	        // This cast is OK even though the jlong might have been read
	        // non-atomically on 32bit systems, since there, one word will
	        // always be zero anyway and the value set is always the same
	        p = (Parker*)addr_from_java(lp);
	      } else {
	        // Grab lock if apparently null or using older version of library
	        MutexLocker mu(Threads_lock);
	        java_thread = JNIHandles::resolve_non_null(jthread);
	        if (java_thread != NULL) {
	          JavaThread* thr = java_lang_Thread::thread(java_thread);
	          if (thr != NULL) {
	            p = thr->parker();
	            if (p != NULL) { // Bind to Java thread for next time.
	              java_lang_Thread::set_park_event(java_thread, addr_to_java(p));
	            }
	          }
	        }
	      }
	    }
	  }

  {- -------------------------------------------
  (1) Parker::unpark() を呼び出す.
      (ついでに (DTrace のフック点)でもある)
    
      (なお, 処理対象のスレッドが見つからなかった場合は, この処理は省略)
      ---------------------------------------- -}

	  if (p != NULL) {
	    HS_DTRACE_PROBE1(hotspot, thread__unpark, p);
	    p->unpark();
	  }
	UNSAFE_END
	
```


