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
JNI_ENTRY(jobject, jni_GetStaticObjectField(JNIEnv *env, jclass clazz, jfieldID fieldID))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("GetStaticObjectField");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE3(hotspot_jni, GetStaticObjectField__entry, env, clazz, fieldID);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	#ifndef JNICHECK_KERNEL
	  DEBUG_ONLY(klassOop param_k = jniCheck::validate_class(thread, clazz);)
	#endif // JNICHECK_KERNEL
	  JNIid* id = jfieldIDWorkaround::from_static_jfieldID(fieldID);
	  assert(id->is_static_field_id(), "invalid static field id");

  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  // Keep JVMTI addition small and only check enabled flag here.
	  // jni_GetField_probe() assumes that is okay to create handles.
	  if (JvmtiExport::should_post_field_access()) {
	    JvmtiExport::jni_GetField_probe(thread, NULL, NULL, id->holder(), fieldID, true);
	  }

  {- -------------------------------------------
  (1) oopDesc::obj_field() でフィールドの値を取得し, その結果を JNI Handle 化しておく.
      ---------------------------------------- -}

	  jobject ret = JNIHandles::make_local(id->holder()->java_mirror()->obj_field(id->offset()));

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, GetStaticObjectField__return, ret);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return ret;
	JNI_END
	
```


