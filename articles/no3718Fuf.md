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
JNI_ENTRY(jobject, jni_NewGlobalRef(JNIEnv *env, jobject ref))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("NewGlobalRef");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, NewGlobalRef__entry, env, ref);

  {- -------------------------------------------
  (1) JNIHandles::resolve() で JNIHandle オブジェクト内の oop を取得し, 
      それを JNIHandles::make_global() で JNIHandles::_global_handles 内に格納する.
      ---------------------------------------- -}

	  Handle ref_handle(thread, JNIHandles::resolve(ref));
	  jobject ret = JNIHandles::make_global(ref_handle);

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, NewGlobalRef__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン.
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


