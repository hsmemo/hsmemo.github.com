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
JNI_ENTRY(jobject, jni_NewLocalRef(JNIEnv *env, jobject ref))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("NewLocalRef");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, NewLocalRef__entry, env, ref);

  {- -------------------------------------------
  (1) JNIHandles::resolve() で JNIHandle オブジェクト内の oop を取得し, 
      それを JNIHandles::make_local() で JNIHandles::_global_handles 内に格納する.
      ---------------------------------------- -}

	  jobject ret = JNIHandles::make_local(env, JNIHandles::resolve(ref));

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, NewLocalRef__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


