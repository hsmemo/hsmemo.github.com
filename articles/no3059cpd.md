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
JNI_ENTRY(jobject, jni_NewObjectA(JNIEnv *env, jclass clazz, jmethodID methodID, const jvalue *args))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("NewObjectA");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, NewObjectA__entry, env, clazz, methodID);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jobject obj = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(NewObjectA, jobject, (const jobject)obj);
	
  {- -------------------------------------------
  (1) alloc_object() をオブジェクトを確保し, それを JNI Handle 化しておく.
      ---------------------------------------- -}

	  instanceOop i = alloc_object(clazz, CHECK_NULL);
	  obj = JNIHandles::make_local(env, i);

  {- -------------------------------------------
  (1) 引数が入った jvalue 配列を JNI_ArgumentPusherArray オブジェクトに詰め直した後, 
      jni_invoke_nonstatic() を呼んで指定されたコンストラクタを呼び出す.
      ---------------------------------------- -}

	  JavaValue jvalue(T_VOID);
	  JNI_ArgumentPusherArray ap(methodID, args);
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_NONVIRTUAL, methodID, &ap, CHECK_NULL);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return obj;
	JNI_END
	
```


