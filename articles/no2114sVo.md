---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Gets an array containing the CPU times consumed by a set of threads
// (in nanoseconds).  Each element of the array is the CPU time for the
// thread ID specified in the corresponding entry in the given array
// of thread IDs; or -1 if the thread does not exist or has terminated.
// If user_sys_cpu_time = true, the sum of user level and system CPU time
// for the given thread is returned; otherwise, only user level CPU time
// is returned.
```

### 名前(function name)
```
JVM_ENTRY(void, jmm_GetThreadCpuTimesWithKind(JNIEnv *env, jlongArray ids,
                                              jlongArray timeArray,
                                              jboolean user_sys_cpu_time))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし, 配列であるべき引数(ids 及び timeArray)が NULL だった場合は, NullPointerException.
      ---------------------------------------- -}

	  // Check if threads is null
	  if (ids == NULL || timeArray == NULL) {
	    THROW(vmSymbols::java_lang_NullPointerException());
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);
	  typeArrayOop ta = typeArrayOop(JNIHandles::resolve_non_null(ids));
	  typeArrayHandle ids_ah(THREAD, ta);
	
	  typeArrayOop tia = typeArrayOop(JNIHandles::resolve_non_null(timeArray));
	  typeArrayHandle timeArray_h(THREAD, tia);
	
  {- -------------------------------------------
  (1) 引数の配列(ids_ah)をチェックしておく.
      (もし不正な値であれば IllegalArgumentException)
      ---------------------------------------- -}

	  // validate the thread id array
	  validate_thread_id_array(ids_ah, CHECK);
	
  {- -------------------------------------------
  (1) もし, 引数で渡された配列(= 返値を入れるべき配列)の長さが
      スレッド数と異なっていれば, IllegalArgumentException.
      ---------------------------------------- -}

	  // timeArray must be of the same length as the given array of thread IDs
	  int num_threads = ids_ah->length();
	  if (num_threads != timeArray_h->length()) {
	    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(),
	              "The length of the given long array does not match the length of "
	              "the given array of thread IDs");
	  }
	
  {- -------------------------------------------
  (1) 引数で渡された配列(ids)中の各スレッドIDについて, 
      os::thread_cpu_time() で CPU 使用時間を取得し, 
      結果を格納する配列(timeArray)内に値を入れていく.
      ---------------------------------------- -}

	  MutexLockerEx ml(Threads_lock);
	  for (int i = 0; i < num_threads; i++) {
	    JavaThread* java_thread = find_java_thread_from_id(ids_ah->long_at(i));
	    if (java_thread != NULL) {
	      timeArray_h->long_at_put(i, os::thread_cpu_time((Thread*)java_thread,
	                                                      user_sys_cpu_time != 0));
	    }
	  }
	JVM_END
	
```


