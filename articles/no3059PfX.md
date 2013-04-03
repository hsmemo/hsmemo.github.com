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
JNI_ENTRY(jobject, jni_AllocObject(JNIEnv *env, jclass clazz))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("AllocObject");
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, AllocObject__entry, env, clazz);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jobject ret = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(AllocObject, jobject, (const jobject&)ret);
	
  {- -------------------------------------------
  (1) alloc_object() をオブジェクトを確保し, それを JNI Handle 化したものをリターンする.
      ---------------------------------------- -}

	  instanceOop i = alloc_object(clazz, CHECK_NULL);
	  ret = JNIHandles::make_local(env, i);
	  return ret;
	JNI_END
	
```


