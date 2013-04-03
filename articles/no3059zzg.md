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
	#define DEFINE_CALLSTATICMETHOD(ResultType, Result, Tag) \
	\
	  DT_RETURN_MARK_DECL_FOR(Result, CallStatic##Result##Method, ResultType);\
	  DT_RETURN_MARK_DECL_FOR(Result, CallStatic##Result##MethodV, ResultType);\
	  DT_RETURN_MARK_DECL_FOR(Result, CallStatic##Result##MethodA, ResultType);\
	\
	JNI_ENTRY(ResultType, \
	          jni_CallStatic##Result##Method(JNIEnv *env, jclass cls, jmethodID methodID, ...)) \
	  JNIWrapper("CallStatic" XSTR(Result) "Method"); \
	\
	  DTRACE_PROBE3(hotspot_jni, CallStatic##Result##Method__entry, env, cls, methodID);\
	  ResultType ret = 0;\
	  DT_RETURN_MARK_FOR(Result, CallStatic##Result##Method, ResultType, \
	                     (const ResultType&)ret);\
	\
	  va_list args; \
	  va_start(args, methodID); \
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherVaArg ap(methodID, args); \
	  jni_invoke_static(env, &jvalue, NULL, JNI_STATIC, methodID, &ap, CHECK_0); \
	  va_end(args); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END \
	\
	JNI_ENTRY(ResultType, \
	          jni_CallStatic##Result##MethodV(JNIEnv *env, jclass cls, jmethodID methodID, va_list args)) \
	  JNIWrapper("CallStatic" XSTR(Result) "MethodV"); \
	  DTRACE_PROBE3(hotspot_jni, CallStatic##Result##MethodV__entry, env, cls, methodID);\
	  ResultType ret = 0;\
	  DT_RETURN_MARK_FOR(Result, CallStatic##Result##MethodV, ResultType, \
	                     (const ResultType&)ret);\
	\
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherVaArg ap(methodID, args); \
	  jni_invoke_static(env, &jvalue, NULL, JNI_STATIC, methodID, &ap, CHECK_0); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END \
	\
	JNI_ENTRY(ResultType, \
	          jni_CallStatic##Result##MethodA(JNIEnv *env, jclass cls, jmethodID methodID, const jvalue *args)) \
	  JNIWrapper("CallStatic" XSTR(Result) "MethodA"); \
	  DTRACE_PROBE3(hotspot_jni, CallStatic##Result##MethodA__entry, env, cls, methodID);\
	  ResultType ret = 0;\
	  DT_RETURN_MARK_FOR(Result, CallStatic##Result##MethodA, ResultType, \
	                     (const ResultType&)ret);\
	\
	  JavaValue jvalue(Tag); \
	  JNI_ArgumentPusherArray ap(methodID, args); \
	  jni_invoke_static(env, &jvalue, NULL, JNI_STATIC, methodID, &ap, CHECK_0); \
	  ret = jvalue.get_##ResultType(); \
	  return ret;\
	JNI_END
	
```


