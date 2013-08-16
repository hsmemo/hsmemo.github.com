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
JVM_ENTRY(void, JVM_SetThreadPriority(JNIEnv* env, jobject jthread, jint prio))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_SetThreadPriority");

  {- -------------------------------------------
  (1) Threads_lock を取得して, OSThread 等のデータが処理の途中で解放されることがないようにしておく.
      ---------------------------------------- -}

	  // Ensure that the C++ Thread and OSThread structures aren't freed before we operate
	  MutexLocker ml(Threads_lock);

  {- -------------------------------------------
  (1) java_lang_Thread::set_priority() を呼んで, 
      対象の java.lang.Thread オブジェクトの priority フィールドに値を記録しておく.
      ---------------------------------------- -}

	  oop java_thread = JNIHandles::resolve_non_null(jthread);
	  java_lang_Thread::set_priority(java_thread, (ThreadPriority)prio);

  {- -------------------------------------------
  (1) Thread::set_priority() を呼んで, 優先度の設定を行う.
      (ただし, 対象のスレッドがまだ開始されていない場合は, この処理は省略する)
      ---------------------------------------- -}

	  JavaThread* thr = java_lang_Thread::thread(java_thread);
	  if (thr != NULL) {                  // Thread not yet started; priority pushed down when it is
	    Thread::set_priority(thr, (ThreadPriority)prio);
	  }
	JVM_END
	
```


