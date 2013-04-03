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
JNI_ENTRY_NO_PRESERVE(jthrowable, jni_ExceptionOccurred(JNIEnv *env))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("ExceptionOccurred");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionOccurred__entry, env);

  {- -------------------------------------------
  (1) jni_check_async_exceptions() を呼び出して
      Asynchronous Exception のチェックをしておく.
      (もし Asynchronous Exception が存在していれば, この中で pending_exception にセットされる)
      ---------------------------------------- -}

	  jni_check_async_exceptions(thread);

  {- -------------------------------------------
  (1) pending_exception フィールドの値を取得し, JNI Handle 化しておく.
      (これが返値になる)
      ---------------------------------------- -}

	  oop exception = thread->pending_exception();
	  jthrowable ret = (jthrowable) JNIHandles::make_local(env, exception);

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionOccurred__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


