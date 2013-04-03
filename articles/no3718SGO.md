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
JNI_ENTRY(jweak, jni_NewWeakGlobalRef(JNIEnv *env, jobject ref))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("jni_NewWeakGlobalRef");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, NewWeakGlobalRef__entry, env, ref);

  {- -------------------------------------------
  (1) JNIHandles::resolve() で JNIHandle オブジェクト内の oop を取得し, 
      それを JNIHandles::make_weak_global() で JNIHandles::_weak_global_handles 内に格納する.
      ---------------------------------------- -}

	  Handle ref_handle(thread, JNIHandles::resolve(ref));
	  jweak ret = JNIHandles::make_weak_global(ref_handle);

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, NewWeakGlobalRef__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


