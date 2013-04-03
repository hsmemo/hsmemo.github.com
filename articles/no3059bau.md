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
JNI_QUICK_ENTRY(jboolean, jni_ExceptionCheck(JNIEnv *env))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("jni_ExceptionCheck");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionCheck__entry, env);

  {- -------------------------------------------
  (1) jni_check_async_exceptions() を呼び出して
      Asynchronous Exception のチェックをしておく.
      (もし Asynchronous Exception が存在していれば, この中で pending_exception にセットされる)
      ---------------------------------------- -}

	  jni_check_async_exceptions(thread);

  {- -------------------------------------------
  (1) pending_exception フィールドが空かどうかを確認しておく.
      (これが返値になる)
      ---------------------------------------- -}

	  jboolean ret = (thread->has_pending_exception()) ? JNI_TRUE : JNI_FALSE;

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, ExceptionCheck__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


