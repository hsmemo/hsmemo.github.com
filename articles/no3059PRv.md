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
JNI_QUICK_ENTRY(void, jni_SetObjectField(JNIEnv *env, jobject obj, jfieldID fieldID, jobject value))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("SetObjectField");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE4(hotspot_jni, SetObjectField__entry, env, obj, fieldID, value);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  oop o = JNIHandles::resolve_non_null(obj);
	  klassOop k = o->klass();
	  int offset = jfieldIDWorkaround::from_instance_jfieldID(k, fieldID);

  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  // Keep JVMTI addition small and only check enabled flag here.
	  // jni_SetField_probe_nh() assumes that is not okay to create handles
	  // and creates a ResetNoHandleMark.
	  if (JvmtiExport::should_post_field_modification()) {
	    jvalue field_value;
	    field_value.l = value;
	    o = JvmtiExport::jni_SetField_probe_nh(thread, obj, o, k, fieldID, false, 'L', (jvalue *)&field_value);
	  }

  {- -------------------------------------------
  (1) oopDesc::obj_field_put() を呼んで, フィールドに値を書き込む.
      ---------------------------------------- -}

	  o->obj_field_put(offset, JNIHandles::resolve(value));

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE(hotspot_jni, SetObjectField__return);
	JNI_END
	
```


