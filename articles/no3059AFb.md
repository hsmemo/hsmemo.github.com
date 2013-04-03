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
JNI_ENTRY(void, jni_CallVoidMethod(JNIEnv *env, jobject obj, jmethodID methodID, ...))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("CallVoidMethod");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, CallVoidMethod__entry, env, obj, methodID);

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_VOID_RETURN_MARK(CallVoidMethod);
	
  {- -------------------------------------------
  (1) 可変長引数を JNI_ArgumentPusherVaArg オブジェクトに詰め直した後, 
      jni_invoke_nonstatic() を呼んで実際の呼び出し処理を行う.
      ---------------------------------------- -}

	  va_list args;
	  va_start(args, methodID);
	  JavaValue jvalue(T_VOID);
	  JNI_ArgumentPusherVaArg ap(methodID, args);
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_VIRTUAL, methodID, &ap, CHECK);
	  va_end(args);
	JNI_END
	
```


