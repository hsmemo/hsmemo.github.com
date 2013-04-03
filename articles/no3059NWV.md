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
JNI_ENTRY(jint, jni_MonitorExit(JNIEnv *env, jobject jobj))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, MonitorExit__entry, env, jobj);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jint ret = JNI_ERR;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(MonitorExit, jint, (const jint&)ret);
	
  {- -------------------------------------------
  (1) ロック開放対象のオブジェクト(jobj)が NULL だった場合は, NullPointerException.
      ---------------------------------------- -}

	  // Don't do anything with a null object
	  if (jobj == NULL) {
	    THROW_(vmSymbols::java_lang_NullPointerException(), JNI_ERR);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle obj(THREAD, JNIHandles::resolve_non_null(jobj));

  {- -------------------------------------------
  (1) ObjectSynchronizer::jni_enter() を呼び出して, ロックの開放を行う.
      ---------------------------------------- -}

	  ObjectSynchronizer::jni_exit(obj(), CHECK_(JNI_ERR));
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  ret = JNI_OK;
	  return ret;
	JNI_END
	
```


