---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp

### 名前(function name)
```
JNI_ENTRY_NO_PRESERVE(void, jni_ExceptionDescribe(JNIEnv *env))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("ExceptionDescribe");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionDescribe__entry, env);

  {- -------------------------------------------
  (1) もしカレントスレッドが pending_exception を持っていれば, 
      以下の if ブロック内で java.lang.Throwable.printStackTrace() を呼び出して出力を行う.
  
      (Throwable でない場合には ". Uncaught exception of type ${クラス名}." といった出力になるようだが, 
       Throwable でない場合とは?? #TODO)
    
      (なお, 例外があっても出力しないケースが 1つだけある.
       例外が ThreadDeath の場合には何も表示しない)
      ---------------------------------------- -}

	  if (thread->has_pending_exception()) {
	    Handle ex(thread, thread->pending_exception());
	    thread->clear_pending_exception();
	    if (ex->is_a(SystemDictionary::ThreadDeath_klass())) {
	      // Don't print anything if we are being killed.
	    } else {
	      jio_fprintf(defaultStream::error_stream(), "Exception ");
	      if (thread != NULL && thread->threadObj() != NULL) {
	        ResourceMark rm(THREAD);
	        jio_fprintf(defaultStream::error_stream(),
	        "in thread \"%s\" ", thread->get_thread_name());
	      }
	      if (ex->is_a(SystemDictionary::Throwable_klass())) {
	        JavaValue result(T_VOID);
	        JavaCalls::call_virtual(&result,
	                                ex,
	                                KlassHandle(THREAD,
	                                  SystemDictionary::Throwable_klass()),
	                                vmSymbols::printStackTrace_name(),
	                                vmSymbols::void_method_signature(),
	                                THREAD);
	        // If an exception is thrown in the call it gets thrown away. Not much
	        // we can do with it. The native code that calls this, does not check
	        // for the exception - hence, it might still be in the thread when DestroyVM gets
	        // called, potentially causing a few asserts to trigger - since no pending exception
	        // is expected.
	        CLEAR_PENDING_EXCEPTION;
	      } else {
	        ResourceMark rm(THREAD);
	        jio_fprintf(defaultStream::error_stream(),
	        ". Uncaught exception of type %s.",
	        Klass::cast(ex->klass())->external_name());
	      }
	    }
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE(hotspot_jni, ExceptionDescribe__return);
	JNI_END
	
```


