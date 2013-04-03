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
	#define DEFINE_GETFIELD(Return,Fieldname,Result) \
	\
	  DT_RETURN_MARK_DECL_FOR(Result, Get##Result##Field, Return);\
	\
	JNI_QUICK_ENTRY(Return, jni_Get##Result##Field(JNIEnv *env, jobject obj, jfieldID fieldID)) \
	  JNIWrapper("Get" XSTR(Result) "Field"); \
	\
	  DTRACE_PROBE3(hotspot_jni, Get##Result##Field__entry, env, obj, fieldID);\
	  Return ret = 0;\
	  DT_RETURN_MARK_FOR(Result, Get##Result##Field, Return, (const Return&)ret);\
	\
	  oop o = JNIHandles::resolve_non_null(obj); \
	  klassOop k = o->klass(); \
	  int offset = jfieldIDWorkaround::from_instance_jfieldID(k, fieldID);  \
	  /* Keep JVMTI addition small and only check enabled flag here.       */ \
	  /* jni_GetField_probe_nh() assumes that is not okay to create handles */ \
	  /* and creates a ResetNoHandleMark.                                   */ \
	  if (JvmtiExport::should_post_field_access()) { \
	    o = JvmtiExport::jni_GetField_probe_nh(thread, obj, o, k, fieldID, false); \
	  } \
	  ret = o->Fieldname##_field(offset); \
	  return ret; \
	JNI_END
	
```


