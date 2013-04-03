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
	#define DEFINE_CALLNONVIRTUALMETHOD(ResultType, Result, Tag) \
	\
	  DT_RETURN_MARK_DECL_FOR(Result, CallNonvirtual##Result##Method, ResultType);\
	  DT_RETURN_MARK_DECL_FOR(Result, CallNonvirtual##Result##MethodV, ResultType);\
	  DT_RETURN_MARK_DECL_FOR(Result, CallNonvirtual##Result##MethodA, ResultType);\
	\
	JNI_ENTRY(ResultType, \
	          jni_CallNonvirtual##Result##Method(JNIEnv *env, jobject obj, jclass cls, jmethodID methodID, ...)) \
	  JNIWrapper("CallNonvitual" XSTR(Result) "Method"); \
	\
	  DTRACE_PROBE4(hotspot_jni, CallNonvirtual##Result##Method__entry, env, obj, cls, methodID);\
	  ResultType ret;\
	  DT_RETURN_MARK_FOR(Result, CallNonvirtual##Result##Method, ResultType, \
	                     (const ResultType&)ret);\
	\
	  va_list args; \
	  va_start(args, methodID); \
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherVaArg ap(methodID, args); \
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_NONVIRTUAL, methodID, &ap, CHECK_0); \
	  va_end(args); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END \
	\
	JNI_ENTRY(ResultType, \
	          jni_CallNonvirtual##Result##MethodV(JNIEnv *env, jobject obj, jclass cls, jmethodID methodID, va_list args)) \
	  JNIWrapper("CallNonvitual" XSTR(Result) "#MethodV"); \
	  DTRACE_PROBE4(hotspot_jni, CallNonvirtual##Result##MethodV__entry, env, obj, cls, methodID);\
	  ResultType ret;\
	  DT_RETURN_MARK_FOR(Result, CallNonvirtual##Result##MethodV, ResultType, \
	                     (const ResultType&)ret);\
	\
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherVaArg ap(methodID, args); \
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_NONVIRTUAL, methodID, &ap, CHECK_0); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END \
	\
	JNI_ENTRY(ResultType, \
	          jni_CallNonvirtual##Result##MethodA(JNIEnv *env, jobject obj, jclass cls, jmethodID methodID, const jvalue *args)) \
	  JNIWrapper("CallNonvitual" XSTR(Result) "MethodA"); \
	  DTRACE_PROBE4(hotspot_jni, CallNonvirtual##Result##MethodA__entry, env, obj, cls, methodID);\
	  ResultType ret;\
	  DT_RETURN_MARK_FOR(Result, CallNonvirtual##Result##MethodA, ResultType, \
	                     (const ResultType&)ret);\
	\
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherArray ap(methodID, args); \
	  jni_invoke_nonstatic(env, &jvalue, obj, JNI_NONVIRTUAL, methodID, &ap, CHECK_0); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END
	
```


