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
JNI_QUICK_ENTRY(jboolean, jni_IsInstanceOf(JNIEnv *env, jobject obj, jclass clazz))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("IsInstanceOf");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, IsInstanceOf__entry, env, obj, clazz);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jboolean ret = JNI_TRUE;

  {- -------------------------------------------
  (1) oopDesc::is_a() を呼んで判定する. 
      ---------------------------------------- -}

	  if (obj != NULL) {
	    ret = JNI_FALSE;
	    klassOop k = java_lang_Class::as_klassOop(
	      JNIHandles::resolve_non_null(clazz));
	    if (k != NULL) {
	      ret = JNIHandles::resolve_non_null(obj)->is_a(k) ? JNI_TRUE : JNI_FALSE;
	    }
	  }

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, IsInstanceOf__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


