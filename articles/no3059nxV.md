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
JNI_QUICK_ENTRY(void, jni_ExceptionClear(JNIEnv *env))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("ExceptionClear");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionClear__entry, env);
	
  {- -------------------------------------------
  (1) カレントスレッドの JvmtiThreadState についてもリセットしておく.
      ---------------------------------------- -}

	  // The jni code might be using this API to clear java thrown exception.
	  // So just mark jvmti thread exception state as exception caught.
	  JvmtiThreadState *state = JavaThread::current()->jvmti_thread_state();
	  if (state != NULL && state->is_exception_detected()) {
	    state->set_exception_caught();
	  }

  {- -------------------------------------------
  (1) カレントスレッドに対して ThreadShadow::clear_pending_exception() を呼び出し, 
      例外を消去する (= pending_exception フィールドをクリアする)
      ---------------------------------------- -}

	  thread->clear_pending_exception();

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE(hotspot_jni, ExceptionClear__return);
	JNI_END
	
```


