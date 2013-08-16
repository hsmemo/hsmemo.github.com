---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp
### 説明(description)
(コメントによると, 以下のように実装を変更するとさらに良くなるのでは? とのこと.

  Threads_lock は性能上重要なロックなので critical section を短くしたい, 
  そのためには, critical section は jthread の resolve と
  Thread->platformevent, Thread->native_thr, Thread->parker 等の取得だけにとどめて, 
  critical section 外で unpark() や thr_kill() を呼ぶように実装を変えた方がいい.
  プラットフォーム依存な箇所を隠蔽するために, critical section 内での処理はクロージャーを返し, 
  critical section 外でそのクロージャーを呼び出すことで処理を行う, という感じにしたらどうか.)


```
// Consider: A better way to implement JVM_Interrupt() is to acquire
// Threads_lock to resolve the jthread into a Thread pointer, fetch
// Thread->platformevent, Thread->native_thr, Thread->parker, etc.,
// drop Threads_lock, and the perform the unpark() and thr_kill() operations
// outside the critical section.  Threads_lock is hot so we want to minimize
// the hold-time.  A cleaner interface would be to decompose interrupt into
// two steps.  The 1st phase, performed under Threads_lock, would return
// a closure that'd be invoked after Threads_lock was dropped.
// This tactic is safe as PlatformEvent and Parkers are type-stable (TSM) and
// admit spurious wakeups.
```

### 名前(function name)
```
JVM_ENTRY(void, JVM_Interrupt(JNIEnv* env, jobject jthread))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_Interrupt");
	
  {- -------------------------------------------
  (1) Threads_lock を取得して, OSThread 等のデータが処理の途中で解放されることがないようにしておく.
      (なお, 処理対象がカレントスレッドの場合には, 必要が無いのでロックは取得しない)
      ---------------------------------------- -}

	  // Ensure that the C++ Thread and OSThread structures aren't freed before we operate
	  oop java_thread = JNIHandles::resolve_non_null(jthread);
	  MutexLockerEx ml(thread->threadObj() == java_thread ? NULL : Threads_lock);

  {- -------------------------------------------
  (1) Thread::interrupt() を呼び出して, 割り込み処理を行う.
      (なお, 上で一度 jthread の resolve をやっているが, ロックを取る前だったのでもう一度やっている)
      (また, 処理対象のスレッドが存在しない場合は, この処理は行わない)
      ---------------------------------------- -}

	  // We need to re-resolve the java_thread, since a GC might have happened during the
	  // acquire of the lock
	  JavaThread* thr = java_lang_Thread::thread(JNIHandles::resolve_non_null(jthread));
	  if (thr != NULL) {
	    Thread::interrupt(thr);
	  }
	JVM_END
	
```


