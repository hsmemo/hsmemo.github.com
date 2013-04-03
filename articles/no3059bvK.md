---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp


### 本体部(body)
```
	#define DEFINE_GETSTATICFIELD(Return,Fieldname,Result) \
	\
	  DT_RETURN_MARK_DECL_FOR(Result, GetStatic##Result##Field, Return);\
	\
	JNI_ENTRY(Return, jni_GetStatic##Result##Field(JNIEnv *env, jclass clazz, jfieldID fieldID)) \
	  JNIWrapper("GetStatic" XSTR(Result) "Field"); \
	  DTRACE_PROBE3(hotspot_jni, GetStatic##Result##Field__entry, env, clazz, fieldID);\
	  Return ret = 0;\
	  DT_RETURN_MARK_FOR(Result, GetStatic##Result##Field, Return, \
	                     (const Return&)ret);\
	  JNIid* id = jfieldIDWorkaround::from_static_jfieldID(fieldID); \
	  assert(id->is_static_field_id(), "invalid static field id"); \
	  /* Keep JVMTI addition small and only check enabled flag here. */ \
	  /* jni_GetField_probe() assumes that is okay to create handles. */ \
	  if (JvmtiExport::should_post_field_access()) { \
	    JvmtiExport::jni_GetField_probe(thread, NULL, NULL, id->holder(), fieldID, true); \
	  } \
	  ret = id->holder()->java_mirror()-> Fieldname##_field (id->offset()); \
	  return ret;\
	JNI_END
	
```


