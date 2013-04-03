---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp
### 説明(description)
// Return the Handle Type


### 名前(function name)
```
JNI_LEAF(jobjectRefType, jni_GetObjectRefType(JNIEnv *env, jobject obj))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("GetObjectRefType");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, GetObjectRefType__entry, env, obj);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jobjectRefType ret;

  {- -------------------------------------------
  (1) 以下の順番で各 JNIHandles 内を調べる.
      どこにも入っていなければ, 返値は JNIInvalidRefType とする.
    
      * JNIHandles::is_local_handle(), JNIHandles::is_frame_handle()
      * JNIHandles::is_global_handle()
      * JNIHandles::is_weak_global_handle()
      ---------------------------------------- -}

	  if (JNIHandles::is_local_handle(thread, obj) ||
	      JNIHandles::is_frame_handle(thread, obj))
	    ret = JNILocalRefType;
	  else if (JNIHandles::is_global_handle(obj))
	    ret = JNIGlobalRefType;
	  else if (JNIHandles::is_weak_global_handle(obj))
	    ret = JNIWeakGlobalRefType;
	  else
	    ret = JNIInvalidRefType;

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, GetObjectRefType__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


