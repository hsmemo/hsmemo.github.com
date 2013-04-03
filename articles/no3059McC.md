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
static void jni_invoke_static(JNIEnv *env, JavaValue* result, jobject receiver, JNICallType call_type, jmethodID method_id, JNI_ArgumentPusher *args, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodHandle method(THREAD, JNIHandles::resolve_jmethod_id(method_id));
	
  {- -------------------------------------------
  (1) (JavaCalls::call() では引数を JavaCallArguments オブジェクトに入れて渡す必要があるので)
      JavaCallArguments オブジェクトを用意し, 
      引数で渡された JNI_ArgumentPusher オブジェクト(args)にセットしておく.
      ---------------------------------------- -}

	  // Create object to hold arguments for the JavaCall, and associate it with
	  // the jni parser
	  ResourceMark rm(THREAD);
	  int number_of_parameters = method->size_of_parameters();
	  JavaCallArguments java_args(number_of_parameters);
	  args->set_java_argument_object(&java_args);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(method->is_static(), "method should be static");
	
  {- -------------------------------------------
  (1) JNI_ArgumentPusher::iterate() を呼んで, 
      JavaCallArguments オブジェクト内に引数を詰め直す.
      ---------------------------------------- -}

	  // Fill out JavaCallArguments object
	  args->iterate( Fingerprinter(method).fingerprint() );

  {- -------------------------------------------
  (1) JavaValue::set_type() を呼んで, 
      引数で渡された JavaValue (result) に返値の型をセットしておく
      ---------------------------------------- -}

	  // Initialize result type
	  result->set_type(args->get_ret_type());
	
  {- -------------------------------------------
  (1) JavaCalls::call() によって, 引数で指定されたメソッドを呼び出す.
      ---------------------------------------- -}

	  // Invoke the method. Result is returned as oop.
	  JavaCalls::call(result, method, &java_args, CHECK);
	
  {- -------------------------------------------
  (1) 返値がポインタ(オブジェクト or 配列)の場合には, JNI Handle 化しておく.
      ---------------------------------------- -}

	  // Convert result
	  if (result->get_type() == T_OBJECT || result->get_type() == T_ARRAY) {
	    result->set_jobject(JNIHandles::make_local(env, (oop) result->get_jobject()));
	  }
	}
	
```


