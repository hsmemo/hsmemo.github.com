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
JNI_ENTRY(void, jni_CallVoidMethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("CallVoidMethodV");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, CallVoidMethodV__entry, env, obj, methodID);

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_VOID_RETURN_MARK(CallVoidMethodV);
	
  {- -------------------------------------------
  (1) 引数が入った va_list を JNI_ArgumentPusherVaArg オブジェクトに詰め直した後, 
      jni_invoke_nonstatic() を呼んで実際の呼び出し処理を行う.
      ---------------------------------------- -}

	  JavaValue jvalue(T_VOID);
	  JNI_ArgumentPusherVaArg ap(methodID, args);
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_VIRTUAL, methodID, &ap, CHECK);
	JNI_END
	
```


