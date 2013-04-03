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
JNI_ENTRY(jclass, jni_GetObjectClass(JNIEnv *env, jobject obj))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("GetObjectClass");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, GetObjectClass__entry, env, obj);

  {- -------------------------------------------
  (1) Klass::java_mirror() で java mirror を取得し, JNI Handle 化しておく.
      ---------------------------------------- -}

	  klassOop k = JNIHandles::resolve_non_null(obj)->klass();
	  jclass ret =
	    (jclass) JNIHandles::make_local(env, Klass::cast(k)->java_mirror());

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, GetObjectClass__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


