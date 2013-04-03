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
JNI_ENTRY(jint, jni_ThrowNew(JNIEnv *env, jclass clazz, const char *message))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("ThrowNew");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, ThrowNew__entry, env, clazz, message);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jint ret = JNI_OK;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(ThrowNew, jint, (const jint&)ret);
	
  {- -------------------------------------------
  (1) THROW_MSG_LOADER_() を用いて, 指定されたクラスの例外オブジェクトを生成し送出する.
      ---------------------------------------- -}

	  instanceKlass* k = instanceKlass::cast(java_lang_Class::as_klassOop(JNIHandles::resolve_non_null(clazz)));
	  Symbol*  name = k->name();
	  Handle class_loader (THREAD,  k->class_loader());
	  Handle protection_domain (THREAD, k->protection_domain());
	  THROW_MSG_LOADER_(name, (char *)message, class_loader, protection_domain, JNI_OK);

  {- -------------------------------------------
  (1) (以下は決して到達しないパス)
      ---------------------------------------- -}

	  ShouldNotReachHere();
	JNI_END
	
```


