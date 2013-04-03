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
// Returns the CPU time consumed by a given thread (in nanoseconds).
// If thread_id == 0, CPU time for the current thread is returned.
// If user_sys_cpu_time = true, user level and system CPU time of
// a given thread is returned; otherwise, only user level CPU time
// is returned.
```

### 名前(function name)
```
JVM_ENTRY(jlong, jmm_GetThreadCpuTimeWithKind(JNIEnv *env, jlong thread_id, jboolean user_sys_cpu_time))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし OS がスレッドの CPU 使用時間に関するアカウンティング機能を提供していなければ, 
      単に -1 をリターンするだけ.
      ---------------------------------------- -}

	  if (!os::is_thread_cpu_time_supported()) {
	    return -1;
	  }
	
  {- -------------------------------------------
  (1) もし, 指定されたスレッドIDが不正な値(0 未満の値)であれば, IllegalArgumentException.
      ---------------------------------------- -}

	  if (thread_id < 0) {
	    THROW_MSG_(vmSymbols::java_lang_IllegalArgumentException(),
	               "Invalid thread ID", -1);
	  }
	
  {- -------------------------------------------
  (1) 以下のどちらかの関数を呼び出して, 引数で指定されたスレッドの CPU 使用時間を取得する.
      * 指定されたスレッドがカレントスレッドである場合
        os::current_thread_cpu_time() を呼び出す.
      * 〃 でない場合
        os::thread_cpu_time() を呼び出す.
  
      (なお, 指定されたスレッドIDに対応するスレッドが見つからない場合は, 単に -1 をリターンする)
      ---------------------------------------- -}

	  JavaThread* java_thread = NULL;
	  if (thread_id == 0) {
	    // current thread
	    return os::current_thread_cpu_time(user_sys_cpu_time != 0);
	  } else {
	    MutexLockerEx ml(Threads_lock);
	    java_thread = find_java_thread_from_id(thread_id);
	    if (java_thread != NULL) {
	      return os::thread_cpu_time((Thread*) java_thread, user_sys_cpu_time != 0);
	    }
	  }
	  return -1;
	JVM_END
	
```


