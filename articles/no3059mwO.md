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
JNI_ENTRY(void, jni_CallStaticVoidMethodV(JNIEnv *env, jclass cls, jmethodID methodID, va_list args))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("CallStaticVoidMethodV");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, CallStaticVoidMethodV__entry, env, cls, methodID);

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_VOID_RETURN_MARK(CallStaticVoidMethodV);
	
  {- -------------------------------------------
  (1) 引数が入った va_list を JNI_ArgumentPusherVaArg オブジェクトに詰め直した後, 
      jni_invoke_static() を呼んで実際の呼び出し処理を行う.
      ---------------------------------------- -}

	  JavaValue jvalue(T_VOID);
	  JNI_ArgumentPusherVaArg ap(methodID, args);
	  jni_invoke_static(env, &jvalue, NULL, JNI_STATIC, methodID, &ap, CHECK);
	JNI_END
	
```


