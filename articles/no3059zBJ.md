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
JNI_ENTRY(void, jni_CallNonvirtualVoidMethodA(JNIEnv *env, jobject obj, jclass cls, jmethodID methodID, const jvalue *args))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("CallNonvirtualVoidMethodA");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE4(hotspot_jni, CallNonvirtualVoidMethodA__entry,
	                env, obj, cls, methodID);

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_VOID_RETURN_MARK(CallNonvirtualVoidMethodA);

  {- -------------------------------------------
  (1) 引数が入った jvalue 配列を JNI_ArgumentPusherArray オブジェクトに詰め直した後, 
      jni_invoke_nonstatic() を呼んで実際の呼び出し処理を行う.
      ---------------------------------------- -}

	  JavaValue jvalue(T_VOID);
	  JNI_ArgumentPusherArray ap(methodID, args);
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_NONVIRTUAL, methodID, &ap, CHECK);
	JNI_END
	
```


