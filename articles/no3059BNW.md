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
JNI_LEAF(jint, jni_GetJavaVM(JNIEnv *env, JavaVM **vm))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("jni_GetJavaVM");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, GetJavaVM__entry, env, vm);

  {- -------------------------------------------
  (1) 引数で渡された JavaVM へのポインタに, main_vm 変数のアドレスをセット.
      ---------------------------------------- -}

	  *vm  = (JavaVM *)(&main_vm);

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, GetJavaVM__return, JNI_OK);

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JNI_OK;
	JNI_END
	
```


