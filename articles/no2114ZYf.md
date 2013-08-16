---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp
### 説明(description)

```
// Support for java.lang.Thread.getStackTrace() and getAllStackTraces() methods
// Return StackTraceElement[][], each element is the stack trace of a thread in
// the corresponding entry in the given threads array
```

### 名前(function name)
```
JVM_ENTRY(jobjectArray, JVM_DumpThreads(JNIEnv *env, jclass threadClass, jobjectArray threads))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_DumpThreads");

  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: JvmtiVMObjectAllocEventCollector)
      ---------------------------------------- -}

	  JvmtiVMObjectAllocEventCollector oam;
	
  {- -------------------------------------------
  (1) もし引数(threads)が NULL であれば, NullPointerException.
      ---------------------------------------- -}

	  // Check if threads is null
	  if (threads == NULL) {
	    THROW_(vmSymbols::java_lang_NullPointerException(), 0);
	  }
	
  {- -------------------------------------------
  (1) もし引数(threads)の配列長が 0 であれば, IllegalArgumentException.
      ---------------------------------------- -}

	  objArrayOop a = objArrayOop(JNIHandles::resolve_non_null(threads));
	  objArrayHandle ah(THREAD, a);
	  int num_threads = ah->length();
	  // check if threads is non-empty array
	  if (num_threads == 0) {
	    THROW_(vmSymbols::java_lang_IllegalArgumentException(), 0);
	  }
	
  {- -------------------------------------------
  (1) もし引数(threads)が java.lang.Thread の配列でなければ, IllegalArgumentException.
      ---------------------------------------- -}

	  // check if threads is not an array of objects of Thread class
	  klassOop k = objArrayKlass::cast(ah->klass())->element_klass();
	  if (k != SystemDictionary::Thread_klass()) {
	    THROW_(vmSymbols::java_lang_IllegalArgumentException(), 0);
	  }
	
  {- -------------------------------------------
  (1) (ResourceMark)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	
  {- -------------------------------------------
  (1) ThreadService::dump_stack_traces() には 
      GrowableArray として引数を渡す必要があるので, 
      引数の配列の中身を GrowableArray に詰め直す.
      ---------------------------------------- -}

	  GrowableArray<instanceHandle>* thread_handle_array = new GrowableArray<instanceHandle>(num_threads);
	  for (int i = 0; i < num_threads; i++) {
	    oop thread_obj = ah->obj_at(i);
	    instanceHandle h(THREAD, (instanceOop) thread_obj);
	    thread_handle_array->append(h);
	  }
	
  {- -------------------------------------------
  (1) ThreadService::dump_stack_traces() を呼んで
      スタックトレースを表す java.lang.StackTraceElement オブジェクト(の配列の配列)を取得し, 
      それを JNIHandle 化してリターンする.
      ---------------------------------------- -}

	  Handle stacktraces = ThreadService::dump_stack_traces(thread_handle_array, num_threads, CHECK_NULL);
	  return (jobjectArray)JNIHandles::make_local(env, stacktraces());
	
	JVM_END
	
```


